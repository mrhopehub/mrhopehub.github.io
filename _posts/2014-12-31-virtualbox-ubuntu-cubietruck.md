---
layout: posts
title: "virtualbox虚拟机下ubuntu10.04建立cubietruck开发环境"
---
# {{ page.title }}
## 1. PL2303驱动安装
<font style="color: red; font-size: 14px;">Windows下:</font>PL2303USB转TTL串口驱动安装，<font style="color: red; font-size: 14px;">安装驱动前需要把任何PL2303设备拔掉，</font>然后运行PL2303_Prolific_DriverInstaller_v1_9_0目录下的PL2303_Prolific_DriverInstaller_v1.9.0.exe安装驱动。最后插入PL2303USB转TTL，windows自动完成驱动查找、安装。
## 2. 虚拟机USB共享驱动安装
<font style="color: red; font-size: 14px;">插入U盘、USB转串口，虚拟设置共享U盘、USB转串口</font>
## 3. 工具链安装
<font style="color: red; font-size: 14px;">Ubuntu</font>工具链安装<xmp class="my_xmp_class">sudo add-apt-repository ppa:linaro-maintainers/toolchain
sudo apt-get update
sudo apt-get install gcc-arm-linux-gnueabi</xmp>
## 4. 安装git、下载uboot源码到$HOME/cubietruck-uboot目录
<xmp class="prettyprint linenums">sudo apt-get install git-core
cd $HOME
mkdir cubietruck-uboot
cd cubietruck-uboot
git clone https://github.com/linux-sunxi/u-boot-sunxi.git</xmp>
## 5. 为了方便编译、写入SD卡、串口调试，笔者使用了三个相应的脚本(cubietruck-uboot目录下)
make脚本
<xmp class="prettyprint linenums">#! /bin/sh

cd u-boot-sunxi
make CROSS_COMPILE=arm-linux-gnueabi- Cubietruck_config
make CROSS_COMPILE=arm-linux-gnueabi-</xmp>
writesdcard脚本
<xmp class="prettyprint linenums">#! /bin/sh

export card=/dev/sdb
export p=""

#If you wish to keep the partition table, run:
sudo dd if=/dev/zero of=${card} bs=1k count=1023 seek=1

#Bootloader
sudo dd if=u-boot-sunxi/spl/sunxi-spl.bin of=${card} bs=1024 seek=8
sudo dd if=u-boot-sunxi/u-boot.img of=${card} bs=1024 seek=40</xmp>
uart脚本
<xmp class="prettyprint linenums">#! /bin/sh

screen /dev/ttyUSB0 115200
</xmp>
<font style="color: red; font-size: 14px;">不要忘了chmod +x</font>
## 6. 编译uboot
<xmp class="prettyprint linenums">cd $HOME/cubietruck-uboot
./make</xmp>
## 7. 烧写到SD卡，插入SD卡，虚拟机设置共享USB
<xmp class="prettyprint linenums">cd $HOME/cubietruck-uboot
./writesdcard</xmp>
<font style="color: red; font-size: 14px;">ubuntu中安全移除SD卡，然后拔出SD卡</font>
## 8. 测试uboot，USB转TTL插入，虚拟机设置共享USB
<xmp class="prettyprint linenums">cd $HOME/cubietruck-uboot
./uart</xmp>
<font style="color: red; font-size: 16px;">不要关闭该窗口，一直作为调试窗口，编译、写SD卡另外打开一个虚拟终端。</font>USB转TTL插入正确连接cubietruck，把SD卡插入cubietruck，上电启动，接收信息
<xmp class="prettyprint linenums">U-Boot SPL 2014.04-10733-gea1ac32 (Dec 31 2014 - 13:35:03)
Board: Cubietruck
DRAM: 2048 MiB
CPU: 960000000Hz, AXI/AHB/APB: 3/2/2
spl: not an uImage at 1600


U-Boot 2014.04-10733-gea1ac32 (Dec 31 2014 - 13:35:03) Allwinner Technology

CPU:   Allwinner A20 (SUN7I)
Board: Cubietruck
I2C:   ready
DRAM:  2 GiB
MMC:   SUNXI SD/MMC: 0
*** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   dwmac.1c50000
Hit any key to stop autoboot:  0 
sun7i# </xmp>