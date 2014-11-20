---
layout: posts
title: "无法找到软件包 libncurses5-dev 的解决方法"
---

# {{ page.title }}
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
1	软件源太少
2	libncurses5-dev	软件名输入错误

推荐几个

网易
deb http://mirrors.163.com/ubuntu/ lucid main universe restricted multiverse
deb-src http://mirrors.163.com/ubuntu/ lucid main universe restricted multiverse
deb http://mirrors.163.com/ubuntu/ lucid-security universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ lucid-security universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ lucid-updates universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ lucid-proposed universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ lucid-proposed universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ lucid-backports universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ lucid-backports universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ lucid-updates universe main multiverse restricted
搜狐
deb http://mirrors.shlug.org/ubuntu/ lucid main universe restricted multiverse
deb-src http://mirrors.shlug.org/ubuntu/ lucid main universe restricted multiverse
deb http://mirrors.shlug.org/ubuntu/ lucid-security universe main multiverse restricted
deb-src http://mirrors.shlug.org/ubuntu/ lucid-security universe main multiverse restricted
deb http://mirrors.shlug.org/ubuntu/ lucid-updates universe main multiverse restricted
deb http://mirrors.shlug.org/ubuntu/ lucid-proposed universe main multiverse restricted
deb-src http://mirrors.shlug.org/ubuntu/ lucid-proposed universe main multiverse restricted
deb http://mirrors.shlug.org/ubuntu/ lucid-backports universe main multiverse restricted
deb-src http://mirrors.shlug.org/ubuntu/ lucid-backports universe main multiverse restricted
deb-src http://mirrors.shlug.org/ubuntu/ lucid-updates universe main multiverse restricted

1 更改软件源
	sudo gedit /etc/apt/sources.list
	把list中的内容替换为上面两部分
2 测试软件源
	点击	系统》》系统管理》》更新管理器
	点击检查	可能显示不可用的软件源，从sources.list删除即可

其他软件源

#srt source
deb http://ubuntu.srt.cn/ubuntu/ lucid main restricted universe multiverse
deb http://ubuntu.srt.cn/ubuntu/ lucid-security main restricted universe multiverse
deb http://ubuntu.srt.cn/ubuntu/ lucid-updates main restricted universe multiverse
deb http://ubuntu.srt.cn/ubuntu/ lucid-proposed main restricted universe multiverse
deb http://ubuntu.srt.cn/ubuntu/ lucid-backports main restricted universe multiverse
deb-src http://ubuntu.srt.cn/ubuntu/ lucid main restricted universe multiverse
deb-src http://ubuntu.srt.cn/ubuntu/ lucid-security main restricted universe multiverse
deb-src http://ubuntu.srt.cn/ubuntu/ lucid-updates main restricted universe multiverse
deb-src http://ubuntu.srt.cn/ubuntu/ lucid-proposed main restricted universe multiverse
deb-src http://ubuntu.srt.cn/ubuntu/ lucid-backports main restricted universe multiverse
#Ubuntu官方上海源 
deb http://mirror.rootguide.org/ubuntu/ lucid main universe restricted multiverse
deb-src http://mirror.rootguide.org/ubuntu/ lucid main universe restricted multiverse
deb http://mirror.rootguide.org/ubuntu/ lucid-security universe main multiverse restricted
deb-src http://mirror.rootguide.org/ubuntu/ lucid-security universe main multiverse restricted
deb http://mirror.rootguide.org/ubuntu/ lucid-updates universe main multiverse restricted
deb http://mirror.rootguide.org/ubuntu/ lucid-proposed universe main multiverse restricted
deb-src http://mirror.rootguide.org/ubuntu/ lucid-proposed universe main multiverse restricted
deb http://mirror.rootguide.org/ubuntu/ lucid-backports universe main multiverse restricted
deb-src http://mirror.rootguide.org/ubuntu/ lucid-backports universe main multiverse restricted
deb-src http://mirror.rootguide.org/ubuntu/ lucid-updates universe main multiverse restricted 

#骨头源，骨头源是bones7456架设的一个Ubuntu源 ，提供ubuntu,deepin 
deb http://ubuntu.srt.cn/ubuntu/ natty main universe restricted multiverse 
deb-src http://ubuntu.srt.cn/ubuntu/ natty main universe restricted multiverse 
deb http://ubuntu.srt.cn/ubuntu/ natty-security universe main multiverse restricted 
deb-src http://ubuntu.srt.cn/ubuntu/ natty-security universe main multiverse restricted 
deb http://ubuntu.srt.cn/ubuntu/ natty-updates universe main multiverse restricted 
deb http://ubuntu.srt.cn/ubuntu/ natty-proposed universe main multiverse restricted 
deb-src http://ubuntu.srt.cn/ubuntu/ natty-proposed universe main multiverse restricted 
deb http://ubuntu.srt.cn/ubuntu/ natty-backports universe main multiverse restricted 
deb-src http://ubuntu.srt.cn/ubuntu/ natty-backports universe main multiverse restricted 
deb-src http://ubuntu.srt.cn/ubuntu/ natty-updates universe main multiverse restricted

#大家可以自己根据自己的版本设置一下，不一定局限于ubuntu 11.04，下面列出一些校内更新源。
 
#电子科技大学
deb http://ubuntu.uestc.edu.cn/ubuntu/ natty main restricted universe multiverse
deb http://ubuntu.uestc.edu.cn/ubuntu/ natty-backports main restricted universe multiverse
deb http://ubuntu.uestc.edu.cn/ubuntu/ natty-proposed main restricted universe multiverse
deb http://ubuntu.uestc.edu.cn/ubuntu/ natty-security main restricted universe multiverse
deb http://ubuntu.uestc.edu.cn/ubuntu/ natty-updates main restricted universe multiverse
deb-src http://ubuntu.uestc.edu.cn/ubuntu/ natty main restricted universe multiverse
deb-src http://ubuntu.uestc.edu.cn/ubuntu/ natty-backports main restricted universe multiverse
deb-src http://ubuntu.uestc.edu.cn/ubuntu/ natty-proposed main restricted universe multiverse
deb-src http://ubuntu.uestc.edu.cn/ubuntu/ natty-security main restricted universe multiverse
deb-src http://ubuntu.uestc.edu.cn/ubuntu/ natty-updates main restricted universe multiverse
#中国科技大学
deb http://debian.ustc.edu.cn/ubuntu/ natty main restricted universe multiverse
deb http://debian.ustc.edu.cn/ubuntu/ natty-backports restricted universe multiverse
deb http://debian.ustc.edu.cn/ubuntu/ natty-proposed main restricted universe multiverse
deb http://debian.ustc.edu.cn/ubuntu/ natty-security main restricted universe multiverse
deb http://debian.ustc.edu.cn/ubuntu/ natty-updates main restricted universe multiverse
deb-src http://debian.ustc.edu.cn/ubuntu/ natty main restricted universe multiverse
deb-src http://debian.ustc.edu.cn/ubuntu/ natty-backports main restricted universe multiverse
deb-src http://debian.ustc.edu.cn/ubuntu/ natty-proposed main restricted universe multiverse
deb-src http://debian.ustc.edu.cn/ubuntu/ natty-security main restricted universe multiverse
deb-src http://debian.ustc.edu.cn/ubuntu/ natty-updates main restricted universe multiverse
</xmp>