---
layout: posts
title: "基于消息的GUI构架"
---

# 基于消息的GUI构架
转载：[http://www.cnblogs.com/lwzz/archive/2012/11/17/2754011.html](http://www.cnblogs.com/lwzz/archive/2012/11/17/2754011.html)<br>
在过去的日子中，大部分编程语言平台的GUI构架几乎没有发生变化。虽然在细节上存在一些差异，比如在功能和编程风格上，但大部分都是采用了相同的构架来响应用户输入以及重新绘制屏幕。这种构架可以被总结为“单线程且基于消息”。
<xmp class="prettyprint linenums">
Message msg;

While(GetMessage(msg))
{
    TranslateMessage(msg);
    DispatchMessage(msg);
}
</xmp>
这段代码可以称为消息循环。在这个循环中，执行顺序是串行的，一个GetMessage只能在前一个GetMessage执行完以后才能执行。

拿WPF或WindowsForm举例，每个线程至少会创建一个拥有消息列队的窗口，并且这个线程的任务之一就是处理列队中的各个消息。只要在应用程序中调用了Application.Run,那么执行这个方法的线程就默认被赋予了这个任务。随后的所有GUI事件，例如用户引发的事件（点击按钮，关闭窗口等），系统引发的事件（重绘窗口，调整大小等），以及应用程序中自定义组件的特定事件等，都将把相应的消息投递给这个消息列队来实现。这意味着，在调用了run之后，随后发生的大部分工作都是由事件处理器为了响应GUI事件而生成的。

 

如图：<br>
<img src="/images/gui架构/gui.jpg" width="800" />
# GUI线程

Gui线程负责取走（get）和分发（dispatch）消息，同时负责描绘界面，如果GUI线程阻塞在分发处理消息这一步的话，那么消息就会在消息队列中积累起来，并等待GUI线程回到消息列队来。

如果阻塞的是一个长时间的操作，比如下载一个文件的话，假设10秒钟，那么用户在10秒钟内都不能进行任何操作，因为线程没法获取新的消息进行处理。

这就是为什么在Windows中存在着MsgWaitForMultipleObjects的原因，这个API使得线程在等待的同时仍然可以运行消息循环。在.NET中，你连这个选择都没有。

消息分发时要考虑到复杂的重入性问题，很难确保一个事件处理器阻塞时，可以安全分发其他GUI事件以响应消息。

因此，一种相对而言更容易掌握的解决方法就是只有在GUI线程中的代码才能够操纵GUI控件，在更新GUI时所需要的其他数据和计算都必须在其他线程中完成，而不是在GUI线程上。如图：
<img src="/images/gui架构/gui+workthread.jpg" width="800" /><br>
通常这意味着把工作转交给线程池完成，然后在得到结果后把结果合并回GUI线程上。