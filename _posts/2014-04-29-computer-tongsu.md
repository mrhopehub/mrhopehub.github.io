---
layout: posts
title: "计算机科学中的通俗说法"
---

# {{ page.title }}
* <font color="blue">CPU控制权转移：</font>
>这样隐藏了很多的信息，但归根结缔还是CS+PC的具体位置，比如说，当CPU控制权转移到一个适当的中断后，这里所说的转移指的是CPU的有一个代码段转移到另一个代码段（中断门确定一个代码段），这里肯定是CS+PC的变化。又比如CPU控制权在操作系统与应用程序之间转移，指的是CPU到底在执行谁的代码，控制权的转移肯定是CS+PC的变化，但此处强调的是CS+PC在谁的执行代码范围之内。又如系统加电后，引导加载程序获得控制权，这里强调的是与将要加载的具体操作系统的关系，其实控制权的转移也是CS+PC的变化，关键是怎样引起变化，在汇编层看的话就是JMP到一个另外一个代码段（可能通过数字指定段描述符），在高级语言层看就是直接调用main函数等等，

* <font color="blue">还不存在堆栈：</font>
>如引导加载程序在系统加电后获得控制权时，还不存在堆栈，指的是栈指针不可用（或者说是任意的位置），并不是不存在SS寄存器。

* <font color="blue">给系统一个个时钟：</font>
>现代的操作系统一般都是建立在中断的基础上的，操作系统在管理应用程序的时间片时要用到定时，定时结束时操作系统就要收回CPU控制权，这就必须通过中断机制，所以要想保证操作系统正常运行，必须对时钟模块进行初始化（时钟频率等等）、设置时钟中断程序等，而各个平台的时钟模块有不同，所以在做系统移植是常说的给系统一个时钟就是这个意思