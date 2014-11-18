---
layout: posts
title: "eclipse 常用"
---

### Process Context Switch in Linux Kernel
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
Basic call path of process scheduling in Linux  (start from kernel/sched.c):
    schedule()->context_switch()->switch_to()->__switch_to()
 
When switching process context, two main parts should be taken into consideration:
switch global page table 
switch kernel stack and hardware context.
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
switch %esp
hardware context switch (__switch_to())
stack switch. 
This piece of code is so excellent and well designed, and the functionalities are clearly described by comments. We should notice that when calling __switch_to(), %[next_ip] is pushed into stack so the %eip of next process is used as __switch_to()'s param. Then we can see the next line ("1:\t") is a symbol, which is used as %[prev_ip] (in line "movl %1f, %[prev_ip]\n\t"). After switching to the new context, %eip is stored here. When new process starts, %ebp is poped and flags are restored. If the process we switch to is a new process, %[next_ip] will be the address of ENTRY(ret_from_fork) (in file arch/x86/kernel/entry_32.S), so we cannot use "call __switch_to" instead of "jmp __switch_to" because "call" will push the address of the following code into stack and we can't get the address of ret_from_fork in %[next_ip]. Here will do "pushl" operation manually.
</xmp>