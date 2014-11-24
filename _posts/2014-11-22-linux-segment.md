---
layout: posts
title: "Linux中的段"
---
# {{ page.title }}
摘抄：陈莉君 《深入分析Linux内核源代码》
## 2.3.7 Linux中的段
<div align=center><img src="/images/linux中的段/图2.9逻辑—线性地址转换.gif" alt="图2.9"></div>
<div align=center>图2.9 逻辑—线性地址转换</div>
<div align=center><img src="/images/linux中的段/图2.10段描述符的一般格式.gif" alt="图2.10"></div>
<div align=center>图2.10 段描述符的一般格式</div>
<xmp class="my_xmp_class">    Intel微处理器的段机制是从8086开始提出的，那时引入的段机制解决了从CPU内部16位地址到20位实地址的转换。为了保持这种兼容性，386仍然使用段机制，但比以前复杂得多。因此，Linux内核的设计并没有全部采用Intel所提供的段方案，仅仅有限度地使用了一下分段机制。这不仅简化了Linux内核的设计，而且为把Linux移植到其他平台创造了条件，因为很多RISC处理器并不支持段机制。但是，对段机制相关知识的了解是进入Linux内核的必经之路。
	从2.2版开始，Linux让所有的进程（或叫任务）都使用相同的逻辑地址空间，因此就没有必要使用局部描述符表LDT。但内核中也用到LDT，那只是在VM86模式中运行Wine，因为就是说在Linux上模拟运行Winodws软件或DOS软件的程序时才使用。
	Linux在启动的过程中设置了段寄存器的值和全局描述符表GDT的内容，段的定义在include/asm-i386/segment.h中：
</xmp>
<xmp class="prettyprint linenums">#define __KERNEL_CS     0x10    ／＊内核代码段, index=2,TI=0,RPL=0＊／

#define __KERNEL_DS     0x18    ／＊内核数据段, index=3,TI=0,RPL=0＊／

#define __USER_CS       0x23    ／＊用户代码段, index=4,TI=0,RPL=3＊／

#define __USER_DS       0x2B    ／＊用户数据段, index=5,TI=0,RPL=3＊／
</xmp>
<xmp class="my_xmp_class">    从定义看出，没有定义堆栈段，实际上，Linux内核不区分数据段和堆栈段，这也体现了Linux内核尽量减少段的使用。因为没有使用LDT，因此，TI=0,并把这4个段都放在GDT中, index就是某个段在GDT表中的下标。内核代码段和数据段具有最高特权，因此其RPL为0，而用户代码段和数据段具有最低特权，因此其RPL为3。可以看出，Linux内核再次简化了特权级的使用，使用了两个特权级而不是4个。

	全局描述符表的定义在arch/i386/kernel/head.S中：
</xmp>
<div style="width: 1024px; height: 368px; overflow: scroll">
<xmp class="prettyprint linenums">ENTRY(gdt_table)
    .quad 0x0000000000000000	/* NULL descriptor */
	.quad 0x0000000000000000	/* not used */
	.quad 0x00cf9a000000ffff	/* 0x10 kernel 4GB code at 0x00000000 */
	.quad 0x00cf92000000ffff	/* 0x18 kernel 4GB data at 0x00000000 */
	.quad 0x00cffa000000ffff	/* 0x23 user   4GB code at 0x00000000 */
	.quad 0x00cff2000000ffff	/* 0x2b user   4GB data at 0x00000000 */
	.quad 0x0000000000000000	/* not used */
	.quad 0x0000000000000000	/* not used */
	/*
	 * The APM segments have byte granularity and their bases
	 * and limits are set at run time.
	 */
	.quad 0x0040920000000000	/* 0x40 APM set up for bad BIOS's */
	.quad 0x00409a0000000000	/* 0x48 APM CS    code */
	.quad 0x00009a0000000000	/* 0x50 APM CS 16 code (16 bit) */
	.quad 0x0040920000000000	/* 0x58 APM DS    data */
	.fill NR_CPUS*4,8,0		/* space for TSS's and LDT's */
	.fill NR_CPUS*4,8,0		/* space for TSS's and LDT's */
	.fill NR_CPUS*4,8,0		/* space for TSS's and LDT's */
</xmp>
</div>
<xmp class="my_xmp_class">    从代码可以看出，GDT放在数组变量gdt_table中。按Intel规定，GDT中的第一项为空，这是为了防止加电后段寄存器未经初始化就进入保护模式而使用GDT的。第二项也没用。从下标2到5共4项对应于前面的4种段描述符值。对照图2.10，从描述符的数值可以得出：
	·段的基地址全部为0x00000000
	·段的上限全部为0xffff
	·段的粒度G为1，即段长单位为4KB
	·段的D位为1，即对这四个段的访问都为32位指令
	·段的P位为1，即四个段都在内存。
	由此可以得出，每个段的逻辑地址空间范围为0～4GB。读者可能对此不太理解，但只要对照图2.9就可以发现，这种设置既简单又巧妙。因为每个段的基地址为0，因此，逻辑地址到线性地址映射保持不变，也就是说，偏移量就是线性地址，我们以后所提到的逻辑地址（或虚拟地址）和线性地址指的也就是同一地址。看来，Linux巧妙地把段机制给绕过去了，而完全利用了分页机制。
	从逻辑上说，Linux巧妙地绕过了逻辑地址到线性地址的映射，但实质上还得应付Intel所提供的段机制。只不过，Linux把段机制变得相当简单，它只把段分为两种：用户态（RPL＝3）的段和内核态（RPL=0）的段，因此，描述符投影寄存器的内容很少发生变化，只在进程从用户态切换到内核态或者反之时才发生变化。另外，用户段和内核段的区别也仅仅在其RPL不同，因此内核根本无需访问描述符投影寄存器，当然也无需访问GDT，而仅从段寄存器的最低两位就可以获取RPL的信息。Linux这样设计所带来的好处是显而易见的，Intel的分段部件对Linux性能造成的影响可以忽略不计。
	在上面描述的GDT表中，紧接着那四个段描述的两个描述符被保留，然后是四个高级电源管理（APM）特征描述符，对此不进行详细讨论。
	按Intel的规定，每个进程有一个任务状态段（TSS）和局部描述符表LDT，但Linux也没有完全遵循Intel的设计思路。如前所述，Linux的进程没有使用LDT，而对TSS的使用也非常有限，每个CPU仅使用一个TSS。
	通过上面的介绍可以看出，Intel的设计可谓周全细致，但Linux的设计者并没有完全陷入这种沼泽，而是选择了简洁而有效的途径，以完成所需功能并达到较好的性能为目
</xmp>
## 学而不思则罔
<xmp class="my_xmp_class">2.3.7节从分段的角度讨论了处理器与内核之间的配合。下面从更高的角度谈一下处理器与内核的配合实现多任务。计算机系统从最初发展到现在，硬件、软件都在发展着，而且二者的发展有些地方是目的相同的。例如多任务这个方向，内存管理是实现多任务的必要条件，硬件在这方面的发展是MMU去支持内存管理，而软件的发展就是内核去配合MMU，从而才能实现内存管理的虚拟空间、分页。
</xmp>
<font color="red">但是有些时候，内核并不用完全配合处理器的机制也能很好的实现多任务，例如内核并没有完全配合i386的分段机制。</font><br>
<xmp class="my_xmp_class">一下从三个方面讨论处理器(i386/32位)与内核的配合：
    1.  内存管理
    2.  中断支持
    3.  任务切换
</xmp>
<font color="red">这三个方面是要实现多任务的必要条件，另外可以从这三个方面很好的去理解i386的寄存器。</font>
<xmp class="my_xmp_class">下面列举一下i386/32位处理器的寄存器：
</xmp>
<div align=center><img src="/images/linux中的段/普通编程寄存器.jpg" alt="普通编程寄存器"></div>
<div align=center>图1 普通编程寄存器</div>
<div align=center><img src="/images/linux中的段/内存管理寄存器.jpg" alt="内存管理寄存器"></div>
<div align=center>图2 内存管理寄存器</div>
<div align=center><img src="/images/linux中的段/控制寄存器.png" alt="控制寄存器"></div>
<div align=center>图3 控制寄存器</div>
<xmp class="my_xmp_class">说明：64位处理器的控制寄存器还有CR8这里并没有列出来，另外CR4在开启PAE模式用到，所以这里只讨论控制寄存器的CR0~CR3。
</xmp>
### 1.内存管理
<xmp class="my_xmp_class">这里主要涉及寄存器GDTR、LDTR、CR0~CR3，其实也就是i386的分段机制与分页机制。由于在很多RISC架构的处理器仅支持分页不支持分段，内核为了移植的方便，所以内核并没有完全配合分段机制。怎么能够躲过内存的分段机制呢，内核把所有段基地址都设置为0，这样逻辑地址和线性地址是一样的(这也是不区分逻辑地址与线性地址的原因)，具体可以参考上文。

下面说一下处理器如何有逻辑地址得到线性地址，当前的指令地址是通过CS/IP组合实现的，而CS是段选择寄存器，IP是偏移寄存器。段选择寄存器用于查找段描述符，段描述符中存放着段基地址，从而段基地址加上偏移量实现逻辑地址到线性地址的转换。那么怎么通过段选择符查找段描述符呢？其实所有的段描述符都放在内存之中的一个表中(虽然段选择符的影子寄存器中存在段选择符)，这样就存在一个问题，这个表的物理地址是什么呢？GDTR就是用来描述这个表的物理地址等属性。

那么LDTR的作用是什么呢？由于Intel在设计处理器的时候考虑到每个任务都用于存放自己私有段的表，同样这个表也放在内存中，LDTR就用于描述这个表的属性，但是大多数用户态的进程并不使用私有描述，所以这里不多介绍。

上面只是分段机制的实现，那么分页机制的实现就要通过CR0~CR3实现，不用多说，需要注意的是处理器的分页机制是通过两级实现。

最后需要注意的两点：
    1.系统调用时通过异常机制实现的，处理器硬件会自动实现CS/IP的切换(也就是处理器自动实现__USER_CS、__USER_DS与__KERNEL_CS、__USER_DS的切换)。内核要做的就是去把设计好的_USER_CS、__USER_DS、__KERNEL_CS、__USER_DS添加到全局描述表中。

    2.由于内核绕开了分段机制仅通过分页机制实现虚拟内存，所以在实现进程切换时，要进行页表的切换，也就是CR0~CR3的切换。
</xmp>
### 2.中断支持
<xmp class="my_xmp_class">不用多说，主要是IDTR实现，内核配合(把设计好的门描述符添加到中断描述附表中)。

需要注意的是：所有的中断发生时，通过IDTR获得中断门段描述符中的代码段选择符都是__KERNEL_CS，不同的是IP不一样。而系统调用的异常门描述符中的代码段选择符也是__KERNEL_CS，而IP指向system_call，system_call通过不同的参数实现不同的系统调用。
</xmp>
### 3.任务切换
主要是TR实现<br><br>
早期的Linux版本利用80x86体系结构所提供的硬件支持，并通过far jmp指令跳到next进程TSS描述符的选择符来执行进程切换(TSS描述符存在全局描述符表中，TR寄存器存放的是TSS描述符的选择符)。但是2.6之后的版本进程切换并没有使用硬件支持，而是内核自己实现了进程的切换。<br>
具体参考：[Process-Context-Switch-in-Linux-Kernel]({% post_url 2014-11-18-Process-Context-Switch-in-Linux-Kernel %})
### 小结
<xmp class="my_xmp_class">上面三个方面讨论了处理器与内核的配合，既可以促进对内核的理解，也促进了对处理器寄存器的理解。所以在分析新处理器上的内核代码时，也可以从上面三个方面分析。例如arm处理器，由于arm处理器架构比较灵活，一般说arm寄存器并没有内存管理相关的寄存器，只说7种模式下的16个寄存器，其实arm的内存管理需要协处理器CP15去支持。另外arm处理器的内存管理也没有段机制，直接就是页机制。
</xmp>