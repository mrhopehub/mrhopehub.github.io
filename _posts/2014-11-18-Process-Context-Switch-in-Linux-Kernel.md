---
layout: posts
title: "Process Context Switch in Linux Kernel"
---

### Process Context Switch in Linux Kernel
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
Basic call path of process scheduling in Linux  (start from kernel/sched.c):
    schedule()->context_switch()->switch_to()->__switch_to()
 
When switching process context, two main parts should be taken into consideration:
	1.switch global page table 
	2.switch kernel stack and hardware context.
These steps are done in context_switch(), switch_to() and __switch_to(). switch_to() switches kernel stack and __switch_to() handles hardware context switch.
 
Here is the source code of switch_to():
</xmp>
<xmp class="prettyprint linenums">
/* 
* Saving eflags is important. It switches not only IOPL between tasks, 
* it also protects other tasks from NT leaking through sysenter etc. 
*/  
#define switch_to(prev, next, last)                    \  
do {                                    \  
    /*                                \ 
     * Context-switching clobbers all registers, so we clobber    \ 
     * them explicitly, via unused output variables.        \ 
     * (EAX and EBP is not listed because EBP is saved/restored    \ 
     * explicitly for wchan access and EAX is the return value of    \ 
     * __switch_to())                        \ 
     */                                \  
    unsigned long ebx, ecx, edx, esi, edi;                \  
                                    \  
    asm volatile("pushfl\n\t"        /* save    flags */    \  
             "pushl %%ebp\n\t"        /* save    EBP   */    \  
             "movl %%esp,%[prev_sp]\n\t"    /* save    ESP   */ \  
             "movl %[next_sp],%%esp\n\t"    /* restore ESP   */ \  
             "movl $1f,%[prev_ip]\n\t"    /* save    EIP   */    \  
             "pushl %[next_ip]\n\t"    /* restore EIP   */    \  
             __switch_canary                    \  
             "jmp __switch_to\n"    /* regparm call  */    \  
             "1:\t"                        \  
             "popl %%ebp\n\t"        /* restore EBP   */    \  
             "popfl\n"            /* restore flags */    \  
                                    \  
             /* output parameters */                \  
             : [prev_sp] "=m" (prev->thread.sp),        \  
               [prev_ip] "=m" (prev->thread.ip),        \  
               "=a" (last),                    \  
                                    \  
               /* clobbered output registers: */        \  
               "=b" (ebx), "=c" (ecx), "=d" (edx),        \  
               "=S" (esi), "=D" (edi)                \  
                                           \  
               __switch_canary_oparam                \  
                                    \  
               /* input parameters: */                \  
             : [next_sp]  "m" (next->thread.sp),        \  
               [next_ip]  "m" (next->thread.ip),        \  
                                           \  
               /* regparm parameters for __switch_to(): */    \  
               [prev]     "a" (prev),                \  
               [next]     "d" (next)                \  
                                    \  
               __switch_canary_iparam                \  
                                    \  
             : /* reloaded segment registers */            \  
            "memory");                    \  
} while (0)
</xmp>
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
We can see 3 things are done in function switch_to(): 
	1.switch %esp
	2.hardware context switch (__switch_to())
	3.stack switch. 
This piece of code is so excellent and well designed, and the functionalities are clearly described by comments. We should notice that when calling __switch_to(), %[next_ip] is pushed into stack so the %eip of next process is used as __switch_to()'s param. Then we can see the next line ("1:\t") is a symbol, which is used as %[prev_ip] (in line "movl %1f, %[prev_ip]\n\t"). After switching to the new context, %eip is stored here. When new process starts, %ebp is poped and flags are restored. If the process we switch to is a new process, %[next_ip] will be the address of ENTRY(ret_from_fork) (in file arch/x86/kernel/entry_32.S), so we cannot use "call __switch_to" instead of "jmp __switch_to" because "call" will push the address of the following code into stack and we can't get the address of ret_from_fork in %[next_ip]. Here will do "pushl" operation manually.
</xmp>
### 小结
哲理性理解：<br>
如前所述，对于每个硬件处理器，仅通过检查栈就可以获得当前正确的进程。早先的Linux版本没有把内核栈与进程描述符放在一起，而是强制引入全局静态变量current来标示正在运行进程的描述符。在多处理器系统上，有必要把current定义为一个数组，每一个元素对应一个可用CPU。<br>
上面这段话总结一下就是，esp代表当前进程。所以esp的切换就是进程的切换，Linux内核进程切换正是这样实现的。如下：<br>
<xmp class="prettyprint linenums">
movl %[next_sp],%%esp
</xmp>
上面这条命令作用是把next->thread.esp装入esp。即进行esp切换，之后的指令已经是在next进程，只不过还要进行硬件上下文的切换（之前的版本由控制器自动实现），所以可以把堆栈切换之后的指令，看做next进程（代替控制器）去完成硬件上下文的切换（因为next进程在内核态，所以可以访问prev与next进程，从而去保存prev进程硬件上下文，恢复自己的硬件上下文）。<br><br>
这里，如果之前B也被switch_to出去过，那么[next_ip]里存的就是下面这个1f的标号，但如果进程B刚刚被创建，之前没有被switch_to出去过，那么[next_ip]里存的将是ret_ftom_fork（参看copy_thread()函数）。这就是这里为什么不用call __switch_to而用jmp，因为call会导致自动把下面这句话的地址(也就是1:)压栈，然后__switch_to()就必然只能ret到这里，而无法根据需要ret到ret_from_fork。<br>