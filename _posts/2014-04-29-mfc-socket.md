---
layout: posts
title: "MFC之socket编程总结：面向对象"
---

# MFC之socket编程总结：面向对象
<img src="/images/mfc-socket/note1.jpg" width="800">
<img src="/images/mfc-socket/note2.jpg" width="800">
<img src="/images/mfc-socket/note3.jpg" width="800">
<img src="/images/mfc-socket/note4.jpg" width="800"><br>
其实上面讨论的CDialog *m_pDlg与m_sListener、m_sConnected相对于面向对象里的类的可见性，还有对象之间的通信或者说发送消息，另外也看到了MyEchoSocket这个类继承与基类CAsyncSocket，这就是面向对象的继承，这里可以看到两点，1，继承到底是怎么用的。2，怎样应用现有的类实现自己的类即怎样用已有的部件改造为自己的部件。<br><br>
<font color="blue">整体总结了两个问题:</font><br>

1. 把问题拆成组件去解决，这就产生了对象之间如何联系（发送消息或者说相互调用）如m_sListener的on_accept调用dialog的on_accept，另外还有对象的可见性。<br>
2. 那么这些组件是否可以由现有的组件去改造（继承等解决方法）