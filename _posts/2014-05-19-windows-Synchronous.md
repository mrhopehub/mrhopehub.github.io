---
layout: posts
title: "windows下多线程同步、互斥"
---
小结：[http://blog.csdn.net/column/details/killthreadseries.html](http://blog.csdn.net/column/details/killthreadseries.html)
# <font color="blue">windows下多线程同步、互斥</font>
主要四种技术——关键段CS、互斥量Mutex、事件Event、信号量Semaphore
## <font color="blue">同步还是互斥</font>

1. 关键段CS、互斥量Mutex都是实现互斥的，类似临界区。
2. 事件Event、信号量Semaphore用于实现同步，类似等待，接口都是WaitForSingleObject/WaitForMultipleObjects，需要注意的是这两个接口还可用于等待其他内核对象（比如Thread、Console input、<font color="red">Mutex</font>等等）。

## <font color="blue">“遗弃”问题</font>
“遗弃”问题就是——占有某种资源的进程意外终止后，其它等待该资源的进程能否感知。

1. 关键段在这个问题上很简单——由于<font color="red">关键段不能跨进程使用，</font>所以关键段不需要处理“遗弃”问题。
2. 互斥量能够处理“遗弃”问题
3. 事件与信号量都无法解决这一情况

## <font color="blue">满足第三方库</font>
像PortAudio开发库的回调模式有可能是在中断中，所以上面说到的同步与互斥就不太实用了，就要采用其他策略。