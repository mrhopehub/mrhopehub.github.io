---
layout: posts
title: "2.5　内核中的并发"
---


# {{ title }}
转载：[http://book.51cto.com/art/201006/207259.htm](http://book.51cto.com/art/201006/207259.htm)<br>
随着多核笔记本电脑时代的到来，对称多处理器（SMP）的使用不再被限于高科技用户。SMP和内核抢占是多线程执行的两种场景。多个线程能够同时操作共享的内核数据结构，因此，对这些数据结构的访问必须被串行化。

接下来，我们会讨论并发访问情况下保护共享内核资源的基本概念。我们以一个简单的例子开始，并逐步引入中断、内核抢占和SMP等复杂概念。
### 2.5.1　自旋锁和互斥体
访问共享资源的代码区域称作临界区。自旋锁（spinlock）和互斥体（mutex，mutual exclusion的缩写）是保护内核临界区的两种基本机制。我们逐个分析。

自旋锁可以确保在同时只有一个线程进入临界区。其他想进入临界区的线程必须不停地原地打转，直到第1个线程释放自旋锁。注意：这里所说的线程不是内核线程，而是执行的线程。

下面的例子演示了自旋锁的基本用法：
<xmp class="prettyprint linenums">
#include <linux/spinlock.h> 
spinlock_t mylock = SPIN_LOCK_UNLOCKED; /* Initialize */  
 
/* Acquire the spinlock. This is inexpensive if there  
 * is no one inside the critical section. In the face of  
 * contention, spinlock() has to busy-wait.  
 */  
spin_lock(&mylock);  
 
/* ... Critical Section code ... */  
 
spin_unlock(&mylock); /* Release the lock */
</xmp>
与自旋锁不同的是，互斥体在进入一个被占用的临界区之前不会原地打转，而是使当前线程进入睡眠状态。如果要等待的时间较长，互斥体比自旋锁更合适，因为自旋锁会消耗CPU资源。在使用互斥体的场合，多于2次进程切换时间都可被认为是长时间，因此一个互斥体会引起本线程睡眠，而当其被唤醒时，它需要被切换回来。

因此，在很多情况下，决定使用自旋锁还是互斥体相对来说很容易：

(1) 如果临界区需要睡眠，只能使用互斥体，因为在获得自旋锁后进行调度、抢占以及在等待队列上睡眠都是非法的；

(2) 由于互斥体会在面临竞争的情况下将当前线程置于睡眠状态，因此，在中断处理函数中，只能使用自旋锁。（第4章将介绍更多的关于中断上下文的限制。）

下面的例子演示了互斥体使用的基本方法：
<xmp class="prettyprint linenums">
#include <linux/mutex.h> 
 
/* Statically declare a mutex. To dynamically  
   create a mutex, use mutex_init() */  
static DEFINE_MUTEX(mymutex);  
 
/* Acquire the mutex. This is inexpensive if there  
 * is no one inside the critical section. In the face of  
 * contention, mutex_lock() puts the calling thread to sleep.  
 */  
mutex_lock(&mymutex);  
 
/* ... Critical Section code ... */  
 
mutex_unlock(&mymutex);      /* Release the mutex */ 
</xmp>
为了论证并发保护的用法，我们首先从一个仅存在于进程上下文的临界区开始，并以下面的顺序逐步增加复杂性：

(1) 非抢占内核，单CPU情况下存在于进程上下文的临界区；

(2) 非抢占内核，单CPU情况下存在于进程和中断上下文的临界区；

(3) 可抢占内核，单CPU情况下存在于进程和中断上下文的临界区；

(4) 可抢占内核，SMP情况下存在于进程和中断上下文的临界区。

<xmp class="my_xmp_class">
旧的信号量接口

互斥体接口代替了旧的信号量接口（semaphore）。互斥体接口是从-rt树演化而来的，在2.6.16内核中被融入主线内核。

尽管如此，但是旧的信号量仍然在内核和驱动程序中广泛使用。信号量接口的基本用法如下：

#include <asm/semaphore.h>  /* Architecture dependent header */  
 
/* Statically declare a semaphore. To dynamically  
   create a semaphore, use init_MUTEX() */  
static DECLARE_MUTEX(mysem);  
 
down(&mysem);    /* Acquire the semaphore */  
 
/* ... Critical Section code ... */  
 
up(&mysem);      /* Release the semaphore */ 
信号量可以被配置为允许多个预定数量的线程同时进入临界区，但是，这种用法非常罕见。
</xmp>
<font color="blue">1.案例1：进程上下文，单CPU，非抢占内核</font>

这种情况最为简单，不需要加锁，因此不再赘述。

<font color="blue">2.案例2：进程和中断上下文，单CPU，非抢占内核</font>

在这种情况下，为了保护临界区，仅仅需要禁止中断。如图2-4所示，假定进程上下文的执行单元A、B以及中断上下文的执行单元C都企图进入相同的临界区。<br>
![](/images/内核中的并发/进程和中断上下文进入临界区.jpg)<br>
由于执行单元C总是在中断上下文执行，它会优先于执行单元A和B，因此，它不用担心保护的问题。执行单元A和B也不必关心彼此会被互相打断，因为内核是非抢占的。因此，执行单元A和B仅仅需要担心C会在它们进入临界区的时候强行进入。为了实现此目的，它们会在进入临界区之前禁止中断：
<xmp class="prettyprint linenums">
Point A：      
  local_irq_disable();  /* Disable Interrupts in local CPU */  
  /* ... Critical Section ...  */  
  local_irq_enable();   /* Enable Interrupts in local CPU */ 
</xmp>
但是，如果当执行到Point A的时候已经被禁止，local_irq_enable()将产生副作用，它会重新使能中断，而不是恢复之前的中断状态。可以这样修复它：
<xmp class="prettyprint linenums">
unsigned long flags;  
 
Point A:  
  local_irq_save(flags);     /* Disable Interrupts */  
  /* ... Critical Section ... */  
  local_irq_restore(flags);  /* Restore state to what it was at Point A */ 
</xmp>
不论Point A的中断处于什么状态，上述代码都将正确执行。

<font color="blue">3.案例3：进程和中断上下文，单CPU，抢占内核</font>

如果内核使能了抢占，仅仅禁止中断将无法确保对临界区的保护，因为另一个处于进程上下文的执行单元可能会进入临界区。重新回到图2-4，现在，除了C以外，执行单元A和B必须提防彼此。显而易见，解决该问题的方法是在进入临界区之前禁止内核抢占、中断，并在退出临界区的时候恢复内核抢占和中断。因此，执行单元A和B使用了自旋锁API的irq变体：
<xmp class="prettyprint linenums">
unsigned long flags;  
 
Point A:  
  /* Save interrupt state.  
   * Disable interrupts - this implicitly disables preemption */  
  spin_lock_irqsave(&mylock, flags);  
 
  /* ... Critical Section ... */  
 
  /* Restore interrupt state to what it was at Point A */  
  spin_unlock_irqrestore(&mylock, flags); 
</xmp>
我们不需要在最后显示地恢复Point A的抢占状态，因为内核自身会通过一个名叫抢占计数器的变量维护它。在抢占被禁止时（通过调用preempt_disable()），计数器值会增加；在抢占被使能时（通过调用preempt_enable()），计数器值会减少。只有在计数器值为0的时候，抢占才发挥作用。

<font color="blue">4.案例4：进程和中断上下文，SMP机器，抢占内核</font>

现在假设临界区执行于SMP机器上，而且你的内核配置了CONFIG_SMP和CONFIG_PREEMPT。

到目前为止讨论的场景中，自旋锁原语发挥的作用仅限于使能和禁止抢占和中断，时间的锁功能并未被完全编译进来。在SMP机器内，锁逻辑被编译进来，而且自旋锁原语确保了SMP安全性。SMP使能的含义如下：
<xmp class="prettyprint linenums">
unsigned long flags;  
 
Point A:  
  /*  
    - Save interrupt state on the local CPU  
    - Disable interrupts on the local CPU. This
implicitly disables preemption.  
    - Lock the section to regulate access by other CPUs  
   */  
  spin_lock_irqsave(&mylock, flags);  
 
  /* ... Critical Section ... */  
 
  /*  
    - Restore interrupt state and preemption to what it  
      was at Point A for the local CPU  
    - Release the lock  
   */  
  spin_unlock_irqrestore(&mylock, flags); 
</xmp>
在SMP系统上，获取自旋锁时，仅仅本CPU上的中断被禁止。因此，一个进程上下文的执行单元（图2-4中的执行单元A）在一个CPU上运行的同时，一个中断处理函数（图2-4中的执行单元C）可能运行在另一个CPU上。非本CPU上的中断处理函数必须自旋等待本CPU上的进程上下文代码退出临界区。中断上下文需要调用spin_lock()/spin_unlock()：
<xmp class="prettyprint linenums">
spin_lock(&mylock);  
 
/* ... Critical Section ... */  
 
spin_unlock(&mylock); 
</xmp>
除了有irq变体以外，自旋锁也有底半部（BH）变体。在锁被获取的时候，spin_lock_bh()会禁止底半部，而spin_unlock_bh()则会在锁被释放时重新使能底半部。我们将在第4章讨论底半部。
<xmp class="my_xmp_class">
-rt树

实时（-rt）树，也被称作CONFIG_PREEMPT_RT补丁集，实现了内核中一些针对低延时的修改。该补丁集可以从www.kernel.org/pub/linux/kernel/projects/rt下载，它允许内核的大部分位置可被抢占，但是用互斥体代替了一些自旋锁。它也合并了一些高精度的定时器。数个-rt功能已经被融入了主线内核。详细的文档见http://rt.wiki.kernel.org/。
</xmp>
为了提高性能，内核也定义了一些针对特定环境的特定的锁原语。使能适用于代码执行场景的互斥机制将使代码更高效。下面来看一下这些特定的互斥机制。

end

<font color="red">1.最后需要注意，自旋锁临界区不要主动放弃CPU，如进行互斥操作导致的进程睡眠等等,否则可能造成死锁。<br>2.上面说的只是单个中断上下文，实际的情况有可能是两个中断上下文（不同类型的中断共享数据），虽然这种情况并不常见。</font>