---
layout: posts
title: "MinGW安装"
---

### MinGW安装
MinGW最近版本的安装使用了mingw-get-setup.exe，当然也可以手动下载各个组件并解压安装。这里主要说一下mingw-get-setup安装方式，由于网络的原因，安装的过程中相当郁闷，整了一个上午才弄安装好。<font color="red">在用MinGW编译一些开源库的时候，如果MinGW都没安装好，在遇见编译错误时，真是相当蛋疼啊，根本不知道是MinGW错误还是开源库没配置好。</font>
#### <font color="blue">下载mingw-get-setup.exe</font>
[点击下载](http://sourceforge.net/projects/mingw/files/latest/download?source=files)，下载链接有可能更新，建议到MinGW官网寻找下载链接。<br>
><font color="red">注意：由于网络的原因需要翻墙，使用了超级大傻瓜。</font>

#### <font color="blue">开始安装</font>
Step2：Download and Set Up MinGW Installation Manager，出现如下图错误：<br>
![error Installation Manager](/images/MinGW install/error_manager_setup.jpg)<br>
><font color="red">注意：默认设置的超级大傻瓜貌似仅支持浏览器翻墙，所以这个mingw-get根本下载不了，改换其他翻墙软件，比如无界浏览、自由门。</font>

正确的step2：<br>
![correct Installation Manager](/images/MinGW install/correct_manager_setup.jpg)<br>
#### <font color="blue">MinGW Installation Manager</font>
建议勾选Basic Setup的所有选项，如图：<br>
![MinGW Installation Manager](/images/MinGW install/MinGW Installation Manager.jpg)<br>
然后菜单Installation-->Apply Changes-->Review Changes-->Apply,然后等待下载，如图：<br>
![Download Package](/images/MinGW install/Download Package.jpg)<br>
<font color="red">同样由于网络的原因，还有代理的原因，下载的过程中可能出现如下的错误：</font><br>
![mingw-get-error](/images/MinGW install/mingw-get-error.jpg)<br>
<font color="red">只需确定即可，在后面会补救的。</font><br>
下载完成之后，可能会出现错误，<font color="red">这也就是代理软件的问题吧，传输出现错误（即使下载中没有明显的弹出错误窗口）。如下图：</font><br>
![Download Package error](/images/MinGW install/Download Package error.jpg)<br>
找出所有ERROR的包，比如上图中的adalib，就把C:\MinGW\var\cache\mingw-get\packages目录下的gcc-ada-4.8.1-4-mingw32-bin.tar.lzma、gcc-ada-4.8.1-4-mingw32-dev.tar.lzma、gcc-ada-4.8.1-4-mingw32-dll.tar.lzma删除掉。而且只留C:\MinGW\var\cache\mingw-get\packages这个目录分支，其他的分支都要删除，而且该目录下的.in-transit目录也要删除。<font color="red">意思是留下的都是正确下载的packages,然后重新开始安装。直到没有ERROR包，记得有一次安装，竟然连grep命令都没有。</font>
#### <font color="blue">设置环境变量</font>
上面的整个过程只是相当于下载、解压，所以就需要手动设置path，只要把C:\MinGW\bin添加到path即可。<br>
<font color="red">从上面也可以看出，卸载的时候，只要从path中删除C:\MinGW\bin，并把C:\MinGW整个目录删除即可。</font>
#### <font color="blue">离线安装</font>
下载地址：[离线安装包](http://cm.baidupcs.com/file/edd629820cb20085214b9c95c33418b4?fid=2015837509-250528-581091062564248&time=1405388278&sign=FDTAXER-DCb740ccc5511e5e8fedcff06b081203-IGFnz09%2B%2BefpBkwdJgQ1JaIUE6w%3D&to=cmb&fm=N,B,G,mn&newver=1&expires=8h&rt=pr&r=593438819&logid=389965092&vuk=2015837509&fn=MinGW%E5%AE%89%E8%A3%85.zip)<br>
关于翻墙软件就不上传了。