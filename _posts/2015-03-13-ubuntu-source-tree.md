---
layout: posts
title: "ubuntu内核源码树"
---
# {{ page.title }}
## ubuntu版本号13.10
## 1.切换到root账户
<xmp class="prettyprint linenums">sudo su</xmp>
## 2.获取内核源码
<xmp class="prettyprint linenums">apt-cache search linux-source
apt-get install linux-source-3.11.0
cd /usr/src
tar -jxvf linux-source-3.11.0.tar.bz2
ln -s linux-source-3.11.0 linux</xmp>
<font style="color: red; font-size: 14px;">linux-source-3.11.0.tar.bz2只是一个符号文件，连接到linux-source-3.11.0目录下的linux-source-3.11.0.tar.bz2。</font>
## 3.配置内核
<xmp class="prettyprint linenums">cp linux-headers-3.11.0-26-generic/.config linux/.config</xmp>
## 4.编译内核、模块
<xmp class="prettyprint linenums">cd linux
make all -j4</xmp>
## 5.安装模块、内核
<xmp class="prettyprint linenums">make modules_install
make install</xmp>
## 6.更新grub启动菜单
<font style="color: red; font-size: 14px;">其实上一步的安装内核已经更新过grub启动菜单</font>
<xmp class="prettyprint linenums">update-grub2</xmp>
## 7.重启机器