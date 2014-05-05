---
layout: posts
title: "linux类系统启动过程"
---

# linux类系统启动过程
从《嵌入式Linux系统基础》中一段话说起：<br>
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
    这里需要牢记的重点是,Linux需要一个文件系统。很多早期的嵌入式操作系统并不需要文件系统，这一点总是会让那些从事将早期的嵌入式操作系统移植到Linux嵌入式操作系统的工程师感到惊奇与困惑。文件系统包含了预先定义的一组系统目录树以及文件，它们都保存到硬盘启动器或者其他媒介之中，Linux内核将其挂载为根文件系统。
</xmp>
<font color="red">为什么早期的工程师对文件系统感到惊奇与困惑呢？</font><br>
### <font color="blue">嵌入式设备的运行</font>

* <font color="blue">无OS设备（或者说裸机设备）</font><br>
顾名思义，指的是没有操作系统的嵌入式设备，更没有进程的概念。系统完全运行在一个main函数（使用C语言时），虽然可以把这个main函数的运行看做是一个进程，但其算不上一个真正的进程。

* <font color="blue">OS设备</font><br>
有了OS的设备，更多的是强调<font color="red">进程、线程，因为OS的内核只是协调进程、方便进程访问硬件的作用，所以系统的功能性代码是进程实现的。</font><br>
问题是怎么创建一个进程呢？下面就比较一下UC/OS与linux。
<blockquote>
1. uc/os
<blockquote>
<xmp class="prettyprint linenums">
INT8U  OSTaskCreate (void (\*task)(void \*pd),void \*pdata,OS_STK \*ptos,INT8U prio)
</xmp>
</blockquote>
用户只需要实现void (*task)(void *pd)即可。<br><br>
2. linux<br>
常用的方式fork,需要注意的是新进程的代码如何实现呢？也就是if(!(pid=fork())){}中怎么实现呢？很明显两种方式:<br>
<blockquote>
1. 直接写代码或者调用其他函数，此种方式跟OSTaskCreate很相似，但是还会涉及到继承的问题。<br>
2. 调用execv等函数，与第一种方式的区别是，execv由一个参数是文件路径，这就是文件系统的作用。
</blockquote>
</blockquote>

* <font color="blue">总结一下：</font><br>
<font color="red">嵌入式系统功能的实现或以裸机代码实现，或以OS中多进程实现，而进程代码可直接代码实现可读取可执行文件。</font>这也就是为什么linux操作系统需要文件系统。另一方面，linux内核启动结束
<xmp class="prettyprint linenums">
    run_init_process("/sbin/init");
    run_init_process("/etc/init");
    run_init_process("/bin/init");
	run_init_process("/bin/sh");
</xmp>
完全可以去掉，直接fork然后在if段中实现相应的进程代码即可。

### <font color="blue">linux类系统启动过程</font>
主要两步：<br>
1.内核启动<br>
2.创建用户进程<br>
第一步的结束就是run_init_process("/sbin/init");之后开始运行用户进程。许多文章之介绍内核启动的过程，<font color="red">往往忽略用户进程，其实用户进程才是主题。</font>读赵炯博士的《Linux内核完全剖析》说到内核先创建进程，读取/etc/rc文件作为sh输入来执行rc脚本，等待进程结束之后再创建进程并open tty0，运行/bin/sh，也就是shell交互界面。<br>
<font color="red">需要注意的是新版本的linux内核只负责创建第一个init进程，之后的工作有init完成，比如涉及到运行级别、启动了相应的服务、然后显示shell或者x11登陆界面。</font>

### <font color="blue">关于android系统</font>
android系统的SDK并不是linux上的libc库、glib库、qt库等等，而是更加封装的java接口，所以android强调的是中间部分。另一方面，android为了实现中间部分，不得不对linux内核进行修改，比如添加的drivers/staging/android/binder.c来实现Binder IPC来支撑android中间部分。具体的移植android参照[这里](http://community.arm.com/groups/android-community/blog/2013/09/18/from-zero-to-boot-porting-android-to-your-arm-platform)