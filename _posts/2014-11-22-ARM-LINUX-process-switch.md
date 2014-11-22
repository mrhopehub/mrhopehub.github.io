---
layout: posts
title: "ARM-LINUX的进程切换"
---
# {{ page.title }}
转：[http://www.2ndmoon.net/weblog/?p=585](http://www.2ndmoon.net/weblog/?p=585)
<xmp class="my_xmp_class">本文主要记录S3C6410/ARM1176JZF-S架构下Linux(kernel 2.6.35)内核如何进行进程切换。

进程切换是操作系统进程调度的基础，首先要能够实现切换，接下来才谈得上“多进程”、“多线程”以及调度算法等更高级的话题。（这里在说“进程切换”的时候提到多线程，并不是把概念搞混淆了。在内核里谈切换的时候，Linux并不区分进程与线程，因为这里只有task，一个进程里如果有多个线程，每一个都是一个task。内核实际上切换的就是task。所以，来自同一个进程的不同线程的task和来自不同进程的task对于内核来说并没有区别。）

Linux进程切换的核心代码是函数context_switch()，此函数的骨干内容如下：
</xmp>
<xmp class="prettyprint linenums">static inline void
context_switch(struct rq *rq, struct task_struct *prev, struct task_struct *next)
{
    switch_mm(oldmm, mm, next);
	switch_to(prev, next, prev);
}
 
#define switch_to(prev,next,last) \
do { \
 last = __switch_to(prev,task_thread_info(prev), task_thread_info(next)); \
} while (0)
</xmp>
<xmp class="my_xmp_class">其中prev是当前进程/切出进程的task_struct指针，next是下一进程/切入进程的task_struct指针。context_switch()主要做两件事情，一件是切换页表，另一件是切换进程上下文。分别由一个函数来实现，下面分别讲解。
</xmp>
# 1.switch_mm
<xmp class="my_xmp_class">switch_mm()的作用是切换切换进程的页表，要做的最重要的事情就是把下一进程的二级页表地址pgd（物理地址）设置到CPU的CP15控制器。进程的页表pgd可以分为两部分来看，0~3G空间部分是用户空间，采用二级映射，每个进程各不相同；3G~4G空间部分是内核空间，采用一级映射，每个进程都相同，其实每个进程的这一块页表内容都是从内核的页表拷贝来的。切换页表的主要目的是切换用户空间的页表，内核空间部分都一样，不需要切换。所以，如果next是一个内核线程的话，并不会调用switch_mm()。

下面是经过简化的switch_mm()汇编代码：
</xmp>
<xmp class="prettyprint linenums">/* r0 = pgd_phys, * r1 = context_id
 */
ENTRY(cpu_v6_switch_mm)
 mov r2, #0
 orr r0, r0, #TTB_FLAGS_UP
 mcr p15, 0, r2, c7, c5, 6 @ flush BTAC/BTB
 mcr p15, 0, r2, c7, c10, 4 @ drain write buffer
 mcr p15, 0, r0, c2, c0, 0 @ write Translation Table Base Register 0
 mcr p15, 0, r1, c13, c0, 1 @ set context ID
 mov pc, lr
ENDPROC(cpu_v6_switch_mm)
</xmp>
<xmp class="my_xmp_class">其中第8行是最核心的一行，它把pgd的值设置给CP15的C2寄存器，C2即是”Translation Table Base Register 0“（地址转换表基地址寄存器）。

switch_mm()调用完之后，用户空间的内容已经是新的进程了，但这时内核空间还属于老的进程，因为CPU还在老进程的内核栈上面运行。下面要做的就是赶紧把内核空间空间也切换到新进程中去，这就是switch_to()所要做的。
</xmp>
# 2.switch_to
<xmp class="my_xmp_class">switch_to()的作用有两个：一是要把当前所运行的进程(切出进程)的现场(包括各个通用寄存器、SP和PC)保存好；二是切换到新进程(切入进程)，即取出此前已保存的新进程的现场，并从上次保存的地方继续运行。注意，这里所说的的现场是内核空间的现场，用户空间的现场在中断刚刚发生时已经保存过。

下面是经过简化的switch_to()汇编代码：
</xmp>
<xmp class="prettyprint linenums">/* r0 = previous task_struct, r1 = previous thread_info, r2 = next thread_info
 */
ENTRY(__switch_to)
/* thread_info + TI_CPU_SAVE hold saved cpu context, registers value is stored */
/* now ip hold the address of the context of previous process */
 add ip, r1, #TI_CPU_SAVE
/* now r3 hold TP value of next process */
 ldr r3, [r2, #TI_TP_VALUE]
/* store current regs to prev thread_info */
 stmia ip!, {r4 - sl, fp, sp, lr} @ Store most regs on
/* store CPU_DOMAIN of next to r6 */
 ldr r6, [r2, #TI_CPU_DOMAIN]
/* set tp value and domain to cp15 */
 mcr p15, 0, r3, c13, c0, 3 @ yes, set TLS register
 mcr p15, 0, r6, c3, c0, 0 @ Set domain register
/* now r4 hold the address of the next context */
 add r4, r2, #TI_CPU_SAVE
/* put next context to registers */
 ldmia r4, {r4 - sl, fp, sp, pc} @ Load all regs saved previously
ENDPROC(__switch_to)
</xmp>
<xmp class="my_xmp_class">我为每一行都加了注释，应该比较容易理解了。

其中第10行和第19行是比较核心的代码，它们分别是保存当前cpu context以及恢复上一次保存的cpu context。这里所说的”上一次“指的是当前进程在上一次处于内核态的时候，当时在离开内核态（切出）的时候，保存了现场。

这里所说的cpu context是由结构体cpu_context所表示的，内容如下。
</xmp>
<xmp class="prettyprint linenums">struct cpu_context_save {
 __u32 r4;
 __u32 r5;
 __u32 r6;
 __u32 r7;
 __u32 r8;
 __u32 r9;
 __u32 sl;
 __u32 fp;
 __u32 sp;
 __u32 pc;
 __u32 extra[2]; /* Xscale 'acc' register, etc */
};
</xmp>
<xmp class="my_xmp_class">在switch_to()的第10行，当前正在运行的SVC模式下的各寄存器（包括r4-r9, sp, lr等等）都被保存起来。

在switch_to()的第19行，r4指向的是下一进程的cpu_context结构地址，这一行执行完后，cpu context中所保存的内容就被读进各个寄存器，sp和pc都被更新，现在CPU已经不在刚刚的那个内核栈上了。

第10行和第19行的寄存器列表有一处区别：第10行的最后一个寄存器是lr，即调用__switch_to()的返回地址；而第19行的最后一个寄存器是pc。这就是说，在切换的时候，当前进程在切回来的时候会从__switch_to()的下一条指令开始执行，这正是内核所需要的。
</xmp>