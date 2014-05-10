---
layout: posts
title: "linux内核同步"
---

# linux内核同步
### <font color="blue">I/O端口的互斥访问</font>
主要使用struct resource *request_region(unsigned long first, unsigned long n, const char *name)之类的函数保证了，内核中只有申请到该资源的驱动访问该IO。否则内核<font color="red">多处访问同一处IO</font>会造成不确定性。
### <font color="blue">驱动的串行化</font>
在I/O端口的互斥行满足之后，如果内核中多处<font color="red">同时调用驱动函数或者多个进程同时访问某个设备文件</font>就相当于内核多处访问同一处IO。所以要保证调用驱动函数的<font color="red">串行化。</font>例如：
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
为了解决多个不同的SPI设备共享SPI控制器而带来的访问冲突，spi_bitbang使用内核提供的工作队列(workqueue)。workqueue是Linux内核中定义的一种回调处理方式。采用这种方式需要传输数据时，不直接完成数据的传输，而是将要传输的工作分装成相应的消息(spi_message)，发送给对应的workqueue，由与workqueue关联的内核守护线程(daemon)负责具体的执行。由于workqueue会将收到的消息按时间先后顺序排列，这样就是对设备的访问严格串行化，解决了冲突。
</xmp>
</font><font size="5" color="blue">分析到进程，也就差不多了。</font>
### <font color="blue">共享数据的同步</font>
操作系统通常讲到的同步多指共享数据的同步，其<font color="red">归根结底是由共享数据的多处访问造成的，临界区：就是访问和操作共享数据的代码段。</font><font size="5" color="blue">注意临界区强调的是共享数据，而不是代码。</font>
### <font color="blue">编程时需要注意的</font>
<font color="red">假设整个系统的同步已经满足，在添加自己的代码时，只要保证自己的数据的同步，并注意系统已有的同步性（如上面说额端口互斥性、驱动串行化，随意写驱动时就不用管上层的同步性），还要注意不要破坏系统的同步性（比如中断例程中不要占用互斥体）。</font>