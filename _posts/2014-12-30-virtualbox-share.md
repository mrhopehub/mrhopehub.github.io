---
layout: posts
title: "virtualbox下Ubuntu10.04共享文件夹"
---
# {{ page.title }}
1. 安装增强功能
2. 设置--》共享文件夹--》固定分配，共享文件夹路径：D:\vbox（任意非空文件夹，方便测试是否共享文件夹成功），共享文件夹名称：vbox，<font style="color: red; font-size: 14px;">不要选择自动挂载</font>
3. 启动虚拟机，Ctrl+Alt+t打开虚拟终端，输入一下命令
<xmp class="prettyprint linenums">cd $HOME
mkdir shared-vbox
sudo mount -t vboxsf vbox shared-vbox
ls shared-vbox</xmp>
卸载命令：sudo umount shared-vbox
4. 自动挂载脚本,新建文件mntshare并输入一下内容
<xmp class="prettyprint linenums">#!/bin/bash
WIN="vbox"    	#windows下的共享目录名
UBU="$HOME/shared-vbox"	#ubuntu下的挂载点
sudo mount -t vboxsf $WIN $UBU</xmp>
添加可执行权限：sudo chmod +x mntshare