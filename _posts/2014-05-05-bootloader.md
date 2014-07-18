---
layout: posts
title: "嵌入式bootloader注意点"
---

# 嵌入式bootloader注意点
### 简介
关于bootloader的作用不用多说，很多文章都有介绍，但很多文章多是说bootloader只是加载内核，即使说到设备的初始化，也并没有详细介绍，导致自己的迷惑，这里就详细说一下设备的初始化。
### 设备的初始化
##### <font color="blue">控制台(console)的初始化</font>

* 注意这里没有采用“串口”的说法，注意他们之间的区别。通常的情况是video framebuffer driver或者a serial driver作为console。arm linux系统通常是a serial driver（貌似android手机使用的是video framebuffer driver）。在bootloader阶段控制台是可交互的。但是对于内核来需要来说bootloader并不需要这么多的工作，<font color="red">只需要完成enabling any hardware power management etc.即可。内核会 automatically detect which serial port it should use for the kernel console。当然也可以通过'console=' parameter指定某个串口初始化为console</font>但是bootloader初始化可交互的console是为了完成bootloader阶段的调试等等目的。<font color="blue" size="5">换个角度看，console是一个虚拟的，最终是要映射到video framebuffer driver或者a serial driver</font>

##### <font color="blue">内存系统初始化</font>

1. 主要设置内存控制器，满足内存芯片的时序、动态刷新等等<br><font color="red">这些工作完全有bootloader完成，内核对这些完全一无所知，内核中的machine_fixup()绝对不是初始化内存系统的correct place，内核肯定也不会做这方面的工作。也就是说内核看到的内存是可用的。</font>
2. 物理内存的配置要传递给内核<br>
主要涉及到两个方面：物理内存的连续性、物理内存大小。bootloader使用ATAG_MEM parameter把内存布局传递给内核。虽然最少的物理内存片段更好，但是物理内存没有必要完全连续的，这时就需要通过Multiple ATAG_MEM blocks把内存布局传递给内核（需要注意的是如果由任意连续的内存段，内核会合并它们）。也可以通过kernel command line的'mem=' parameter，同样也可以处理不连续的多物理内存段。

##### <font color="blue">启动设备的初始化</font>

* 比如内核镜像在nand flash的话就初始化nand flash。主要的作用是加载内核镜像。

##### <font color="blue">CPU的初始化</font>

* 其实对CPU的要求算不上初始化的部分，只是很多文章都提到了，控制权转给内核时寄存器的要求，所以这里说一下。<br><br>
The CPU must be in SVC (supervisor) mode with both IRQ and FIQ interrupts disabled.<br>
The MMU must be off, i.e. code running from physical RAM with no translated addressing.<br>
Data cache must be off<br>
Instruction cache may be either on or off<br>
CPU register 0 must be 0<br>
CPU register 1 must be the ARM Linux machine type<br>
CPU register 2 must be the physical address of the parameter list<br><br>
注意上面是进入内核镜像（arch/$(ARCH)/boot/head.S）时的要求，并不是vmlinux时的要求。