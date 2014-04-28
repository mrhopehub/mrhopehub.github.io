---
layout: posts
title: "ubuntu12.04编译android最新版本"
---

# ubuntu12.04编译android最新版本
之所以写这篇文章是因为不一样的android版本编译需要不一样的环境，在阅读官方的教程时，对版本的的主线没把握住，导致有些地方不是很理解，所以在这里说一下环境搭建时与版本相关的几点问题还有环境搭建时遇到的问题<br><br>
<font color="red">Android版本：最新版本即master分支</font><br><br>
环境搭建：<br>
<font color="blue">1.操作系统安装：</font><br>
因为要编译最新版本的android，也就是官方所说的master分支，所以对操作系统的要求是64位ubuntu12.04，这里要说一下，虽然官方的教程说64位ubuntu10.04，但是那是编译旧版本android的要求。<br><br>
<font color="blue">2.JDK的安装：</font>
按照官网上的办法安装失败，建议自己下载java安装<br><br>
<font color="blue">3.库的安装：</font>
sudo apt-get install git gnupg flex bison gperf build-essential \<br>
  zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \<br>
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \<br>
  libgl1-mesa-dev g++-multilib mingw32 tofrodos \<br>
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386<br>
出错：<br>
libgl1-mesa-glx:i386 : Depends: libglapi-mesa:i386 (= 8.0.4-0ubuntu0.7)<br>
Recommends: libgl1-mesa-dri:i386 (>= 7.2)<br>
解决办法：<br>
libgl1-mesa-glx:i386改为libglapi-mesa-lts-saucy:i386<br>
<font color="red">另外若把libgl1-mesa-glx:i386改为libgl1-mesa-dri:i386系统重启后could not write bytes：broken pipe</font><br><br>
<font color="blue">4.设置缓冲：</font><br>
prebuilts/misc/linux-x86/ccache/ccache -M 50G<br>
官方说的prebuilt/linux-x86/ccache/ccache -M 50G是4.0.x和之前的老版本的<br>
另外还需要注意，设置一次即可（永久有效），还有该命令的当前目录是源代码的根目录，所以该命令在下载好源码之后执行<br>
官网原文：<br>
<font color="red">
The suggested cache size is 50-100GB. You will need to run the following command once you have downloaded the source code:</font><br><br>
<font color="blue">5.Repo的初始化：</font><br>
repo init -u https://android.googlesource.com/platform/manifest<br>
官方所说的：<br>
repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1<br>
是选择指定的分支（也就是版本）。<br>
repo init -u https://android.googlesource.com/platform/manifest这样是选择master分支，即最新版本<br><br>
<font color="blue">6.repo sync中断：</font><br>
脚本解决：<br>
\#! /bin/bash<br>
echo "=====start repo sync======"<br>
~/bin/repo sync<br>
while [ $? = 1 ]; do <br>
echo “======sync failed, re-sync again======”<br>
sleep 3<br>
~/bin/repo sync<br>
done<br>