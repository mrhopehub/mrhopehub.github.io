---
layout: posts
title: "(转)Java GUI编程SwingUtilities.invokeLater作用"
---

# (转)Java GUI编程SwingUtilities.invokeLater作用
Swing图型界面多线程编程过程中的一些误区<br>
<br>
很多学JAVA程序员都是从Swing开始的，但很多人对AWT GUI线程的机制并没有太深的了解，或者说一直都只了解线程的概念，而不了解AWT对线程的使用。我发现很多人碰到线程阻塞的问题，就通过调用SwingUtilities.invokeLater()来解决。<br>
<br>
其实这是很容易造成误会的地方：

1. 不要以为Swing 是多线程的，实际上Swing 的UI是单线程的
2. 不要以为SwingUtilities.的两个invoke是多线程，实际上它还是单线程的
3. 不要以为invokeLater的意思是当前线程执行完再执行目标线程；以为invokeAndWait的意思是等待目标线程执行完再执行当前线程，实际上压根就不是那么回事

问题代码1：大意是在按下某个按钮的时候调用一个远程服务
<xmp class="prettyprint linenums">
JButton button = new JButton();   
    button.addActionListener(new ActionListener() {   
        @Override  
        public void actionPerformed(ActionEvent e) {   
        invokeRemoteService();// 可能需要等待  
    }   
}); 
</xmp>
在swing系统中，有一个顶级的java.awt.Container(可能是一个JFrame或JDialog实例)，负责启动一个EventDispatchThread线程，单线程，这个线程是负责处理UI事件的。<br>
首先，界面Swing控件向EventDispatchThread的EventQueue提交一个event，由EventDispatchThread负责调度各个event的执行。例如,按下一个JButton的时候，JButton向EventQueue执行postEvent，提交一个ActionEvent。EventDispatchThread线程根据调度算法执行到该event的时候，会调用JButton上的processActionEvent，JButton再调用actionPerformed，这过程并没有执行任何new Thread().start()代码，也就是说JButton的ActionListener.actionPerformed()中的代码完全是在EventDispatchThread线程内执行的。所以，假如我们在任何ActionListener、MouseListener等对象中编写耗时的逻辑，那么整个Swing系统就会出现响应迟钝的现象，更有甚者，如果在这些Listener中执行线程wait()，以等待另一个线程的锁定资源或计算结果，那么实际上就是EventDispatchThread线程被阻塞，整个系统界面就会处于无响应状态，一点反应都没有。<br>
以上是误解造成的，了解这个过程，就很容易看出上面这段代码的问题是什么原因了。解决的方法也倒比较简单，直接new Thread().start();就可以保证EventDispatchThread执行到当前方法的时候快速返回，以便可以去响应来自用户界面的其他事件。<br>
<br>
问题代码2：大意是在按下某个按钮的时候调用一个远程服务，同时处理其他事情
<xmp class="prettyprint linenums">
JButton button = new JButton();   
button.addActionListener(new ActionListener() {   
    @Override  
    public void actionPerformed(ActionEvent e) {   
        // 位置A  
        SwingUtilities.invokeLater(new Runnable() {   
            public void run() {   
                // 位置B  
                invokeRemoteService();// 可能需要等待  
            }   
        });   
        doOtherThing();   
    }   
});
</xmp>
这段代码跟第一段代码唯一的差别是doOtherThing()在invokeRemoteService ()完成之前就能够得到执行，所以造成了invokeRemoteService ()/doOtherThing()好像是在两个线程里执行的假象。实际上invokeLater是把目标代码打包成一个Event提交到EventQueue去了，等到EventDispatchThread线程执行完当前代码段的doOtherThing()后，再去执行这个EventQueue中的Event，这时候就会执行到这个invokeRemoteService ()方法。但是，实际上这两个方法都是在EventDispatchThread中执行的，并没有任何其他Thread来执行。于是，问题1的问题还是没解决。实际上直接new Thread().start()方法就可以了，使用SwingUtilities完全是由于误解造成的滥用。测试方法，在位置A和位置B都加上下面这行代码：<br>
<xmp class="prettyprint linenums">
System.out.println(Thread.currentThread().getId() + Thread.currentThread().getName());
</xmp>
返回的结果都是一样的：<br>
21AWT-EventQueue-0<br>
21AWT-EventQueue-0<br>
[讨论]<br>
一般情况下（除了系统启动时后台创建的Daemon线程），系统的所有执行功能逻辑和业务逻辑的线程都应该是从界面操作触发的。我们应该清楚哪些需要或应该放到EventDispatchThread中去执行，哪些需要或应该创建一个新线程去执行，也需要清醒的知道自己当前编写的是属于什么逻辑。<br>
这个问题我觉得应该把代码分成3层，第一层，UI层，包括UI控件上的Listener逻辑，这是应该给EventDispatchThread去执行的，必须简短高效，快速return；这一层做不完的事情通过new Thread().start()交给下一层去做，我称之为控制层；然后控制层再去调用具体的业务代码，即第三层，业务层。所有由UI控件触发的逻辑都应该这么分。<br>
另一个问题是，Swing并不推荐在EventDispatchThread之外修改界面,那么，如果我们在业务层需要repaint某个控件，或者updateUI应该怎么办呢，那就可以使用SwingUtilities来处理了，这才是正确使用SwingUtilities的场景，也是设计这个工具的目的。