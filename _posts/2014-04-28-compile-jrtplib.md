---
layout: posts
title: "windows下编译jrtplib-3.9.1和jthread-1.3.1"
---

# {{ page.title }}
<font color="blue">第一步，先下载三个必要的文件</font>
<blockquote>
1. 下载 jrtplib-3.9.1：<br>
2. 下载 jthread-1.3.1:<br>
3. 下载cmake-2.8.8-win32-x86.exe程序：
</blockquote>
<font color="blue">
第二步,在windows中安装好cmake，并生成 jrtplib 和 jthread 的VS2008的项目文件</font><br><br>
<font color="blue">
第三步，生成jthread项目</font><br>
<blockquote>
1. 把生成的 jthread_d.lib 拷贝到C:\Program Files\jthread\lib目录下。<br><br>
2. 把jthread-1.3.1\src下的jmutex.h和jthread.h以及jthread的Vs项目下的src\jthreadconfig.h<br>
拷贝到jrtplib-3.9.1\src\jthread目录下
</blockquote>
<font color="blue">
第四步，生成jrtplib项目</font>