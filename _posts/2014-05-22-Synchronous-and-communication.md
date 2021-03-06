---
layout: posts
title: "多线程通信与同步"
---

#多线程通信与同步
## <font color="blue">介绍</font>
前面已经说过线程之间的同步，但是在多线程的设计上思路总是不够清晰，不能直接使用前面提到的线程之间的同步。<font color="red">其实在考虑多线程时首先考虑的是通信问题，没有通信哪来的同步（回想一下串口通信，不用需要串口通信的话，还用考虑同步通信异步通信的问题吗）。</font>
## <font color="blue">与I/O方式的比较</font>
比较不是很贴切，两者也没什么关系，自我感觉还是有点相似性。假设有两个线程A、B，A是界面线程，B是工作者线程（在循环printf），线程A如何控制线程B的暂停/开始呢<font color="red">（前提不结束线程B、不暂停线程B）</font>？通过全局变量循环标志的方法（只不过此时需要线程A设置标志，而线程B需要一直检查标志）,或者通过类似UNIX信号的方法处理了<font color="red">（需要注意此时虽然切换到了线程B的信号处理程序，但并没有改变循环的printf，所以还是需要线程B的循环里检查循环标志）。</font><font color="blue">这两种方法跟I/O方式的中断、轮询很相似。</font>再引一段不知是否权威的话：
<xmp class="my_xmp_class">
信号是进程间通信机制中唯一的异步通信机制，可以看作是异步通知，通知接收信号的进程有哪些事情发生了。信号机制经过POSIX实时扩展后，功能更加强大，除了基本通知功能外，还可以传递附加信息。
</xmp>
## <font color="blue">对通信的理解</font>
<font color="red">最一般的CPU跟外设通信，一般传递的也就是命令、数据，所以线程间的通信大致也分为命令、数据。是命令还是数据看程序怎么处理了（它们都是字节组成的，例如定义里一个int型全局变量command，可以看成命令也可以看成数据。同样socket传输的都是字节，但进程可以把它解释为普通的数据或者命令）。站在这个角度看的话，每个线程可以看做是一个集成电路的一个模块，线程之间的通信看做模块之间的通信，比如定义了全局变量command、data就相当于两个模块之间的命令通信（总线）、数据通信（总线）。</font>
## <font color="blue">线程通信的方法</font>
前面也说了大致分为中断方式通信、轮询方式通信，<font color="red">所以这里主要说的是轮询通信方式。</font>

1. 使用全局变量进行通信
2. 使用自定义消息

这里没有把event看做是通信方法，更多的是把它看做同步方式。<font color="red" size="5">同步的方法多用在全局变量通信的方法上，介绍里的“先通信后同步”。</font>当然不太严格的通信定义(线程（函数）可以访问的资源)远不止这两种方法，因为线程的资源远不止全局变量、消息（例如AfxBeginThread参数中的LPVOID lParam也可以看做是线程的通信，如果传入的HWND，就能获取到hdc，另外还有portaudio的回调函数的userdata等等）。