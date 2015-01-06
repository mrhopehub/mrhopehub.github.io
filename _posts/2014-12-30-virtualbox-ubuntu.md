---
layout: posts
title: "virtualbox搭建ubuntu10.04环境"
---
# {{ page.title }}
准备软件：VirtualBox.exe、ubuntu-10.04-desktop-i386.iso
## 安装VirtualBox
建议到[https://www.virtualbox.org/wiki/Download_Old_Builds](https://www.virtualbox.org/wiki/Download_Old_Builds)下载VirtualBox 4.3.0，<font style="color: red; font-size: 14px;">笔者下载了五六个版本安装之后，新建虚拟机成功，总是在启动虚拟机的时候提示各种各样的错误，所以这里指定一下版本。</font>
## 新建虚拟机、配置虚拟机
1. 新建虚拟机选择linux类的Ubuntu、内存1024MB、磁盘50GB、动态增长
2. 设置虚拟机网络连接方式为桥接网卡

## 为虚拟机安装系统
1. 设置虚拟机存储添加光盘
2. 启动虚拟机安装系统
3. <font style="color: red; font-size: 14px;">Ctrl+Alt+t打开虚拟终端，输入sudo umount -l /isodevice</font>
4. 双击桌面install Ubuntu图标安装系统

## 快速设置Ubuntu10.04
1. 输入法设置（如果刚才是联网安装系统的话，可以跳过此步），需要电脑联网，系统--》系统管理--》语言支持，然后进行语言包安装。<font style="color: red; font-size: 14px;">安装完成之后需要重启生效(建议完成第二步再重启)，重启之后Ctrl+space切换中英输入法。</font>
2. 安装增强功能，设备--》共享粘贴板（双向）、设备--》拖放（双向），虚拟机设备--》安装增强功能、完成后控制--》正常关机，然后重新启动虚拟机。
3. 简单设置Ubuntu桌面，上下面板交换（按住Alt键即可拖动面板）、固定虚拟机小工具栏，<font style="color: red; font-size: 14px;">这样方便虚拟机的全屏切换与退出</font>