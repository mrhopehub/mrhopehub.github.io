---
layout: posts
title: "ubuntu10.04制作cubietruck可启动SD卡"
---
# {{ page.title }}
## 创建工程目录
<xmp class="prettyprint linenums">cd $HOME
mkdir cubietruck-build
cd cubietruck-build/
mkdir cubietruck-uboot
mkdir cubietruck-kernel
mkdir cubietruck-sdcard</xmp>
cubietruck-build为总目录，cubietruck-uboot用于编译uboot，cubietruck-kernel用于编译内核，cubietruck-sdcard用于将编译结果、根文件系统写入SD卡。
## 工具链安装
<xmp class="my_xmp_class">工具链安装，https://releases.linaro.org/13.04/components/toolchain/binaries选择gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.bz2下载到$HOME目录下。profile文件最后添加一行：
export PATH=/opt/gcc-linaro-arm-linux-gnueabihf/gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux/bin:$PATH</xmp><xmp class="prettyprint linenums">sudo mkdir /opt/gcc-linaro-arm-linux-gnueabihf
sudo tar -xjvf gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.bz2 -C /opt/gcc-linaro-arm-linux-gnueabihf/
sudo gedit /etc/profile
source /etc/profile</xmp>
## 其他工具、开发库安装
<xmp class="prettyprint linenums">sudo apt-get install git-core

sudo apt-get install libncurses5-dev

sudo apt-get install libusb-1.0-0-dev
export PKG_CONFIG_PATH=/usr/lib/pkgconfig:$PKG_CONFIG_PATH
sudo pkg-config libusb-1.0 --cflags --libs</xmp>
libncurses5-dev为menuconfig需要的库，libusb-1.0-0-dev为编译bin2fex、fex2bin需要的库
## 获取源码
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-uboot
git clone https://github.com/linux-sunxi/u-boot-sunxi.git
cd $HOME/cubietruck-build/cubietruck-kernel
git clone -b sunxi-3.4 https://github.com/linux-sunxi/linux-sunxi.git
git clone https://github.com/cubieboard/cubie_configs
git clone git://github.com/linux-sunxi/sunxi-tools.git</xmp>
## 编译Uboot
修改$HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi/include/configs/sunxi-common.h文件，去掉以下两块代码
<xmp class="prettyprint linenums">      "fatload $device $partition $scriptaddr ${bootscr}" \
      " || " \
	  "ext2load $device $partition $scriptaddr boot/${bootscr}" \
	  " ||" \</xmp>
第二块
<xmp class="prettyprint linenums">      "fatload $device $partition $scriptaddr ${bootenv}" \
	  " || " \
	  "ext2load $device $partition $scriptaddr boot/${bootenv}" \
	  " || " \</xmp>
开始编译
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi
make CROSS_COMPILE=arm-linux-gnueabihf- Cubietruck_config
make CROSS_COMPILE=arm-linux-gnueabihf-</xmp>
## 编译内核
<xmp class="prettyprint linenums">export PATH=$HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi/tools:$PATH
cd $HOME/cubietruck-build/cubietruck-kernel/linux-sunxi
rm -rv .config
cp ../cubie_configs/kernel-configs/3.4/cubietruck_defconfig .config
make ARCH=arm menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage modules</xmp>
## 编译sunxi-tools
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-kernel/sunxi-tools
make</xmp>
## 构建script.bin
复制$HOME/cubietruck-build/cubietruck-kernel/cubie_configs/sysconfig/linux/目录下的cubietruck.fex到$HOME/cubietruck-build/cubietruck-kernel，修改screen0_output_type为4（VGA显示）。
构建script.bin
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-kernel
rm script.bin
./sunxi-tools/fex2bin cubietruck.fex script.bin</xmp>
## 构建boot.scr
在$HOME/cubietruck-build/cubietruck-kernel/目录下新建boot.cmd，添加以下内容
<xmp class="prettyprint linenums">setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10
ext2load mmc 0 0x43000000 script.bin
ext2load mmc 0 0x48000000 uImage
bootm 0x48000000</xmp>
构建boot.scr
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-kernel
export PATH=$HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi/tools:$PATH
rm boot.scr
mkimage -C none -A arm -T script -d boot.cmd boot.scr</xmp>
## uEnv.txt
在$HOME/cubietruck-build/cubietruck-kernel/目录下新建uEnv.txt，这个文件的作用是存放uboot的环境变量，这里暂时不添加任何内容。
## 制作根文件系统
这里并没有自己编译根文件系统，而是下载别人制作好的根文件系统。可以到[https://releases.linaro.org](https://releases.linaro.org)下载，这里选择https://releases.linaro.org/13.04/ubuntu/quantal-images/developer/linaro-quantal-developer-20130422-342.tar.gz。下载好之后放在$HOME/cubietruck-build/cubietruck-sdcard目录下，用以下命令解压
<xmp class="prettyprint linenums">export ROOTFS_TARBALL=linaro-quantal-developer-20130422-342.tar.gz
cd $HOME/cubietruck-build/cubietruck-sdcard
sudo rm -rv binary/
sudo tar -zxvf ${ROOTFS_TARBALL}</xmp>
$HOME/cubietruck-build/cubietruck-sdcard/binary目录下为根文件系统
## SD卡分区
SD卡为sdb，中间需要输入分区参数，输入内容为(回车代表回车键)：--》c--》u--》n--》p--》1--》回车--》+16M--》n--》p--》2--》回车--》回车--》w
<xmp class="prettyprint linenums">export card=/dev/sdb
export p=""
sudo umount ${card}?
sudo dd if=/dev/zero of=${card} bs=1M count=1
sudo fdisk ${card}
sudo mkfs.ext2 ${card}${p}1
sudo mkfs.ext4 ${card}${p}2</xmp>
## 写入Uboot
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-uboot
export card=/dev/sdb
export p=""
sudo dd if=/dev/zero of=${card} bs=1k count=1023 seek=1
sudo dd if=u-boot-sunxi/u-boot-sunxi-with-spl.bin of=${card} bs=1024 seek=8</xmp>
## 复制启动分区
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-sdcard
export card=/dev/sdb
export p=""
sudo umount ${card}${p}1
sudo mount ${card}${p}1 /mnt
sudo rm -rv /mnt/*
sudo cp -v ../cubietruck-kernel/uEnv.txt /mnt
sudo cp -v ../cubietruck-kernel/boot.scr /mnt
sudo cp -v ../cubietruck-kernel/script.bin /mnt
sudo cp -v ../cubietruck-kernel/linux-sunxi/arch/arm/boot/uImage /mnt
sudo umount ${card}${p}1</xmp>
## 复制根文件系统
<xmp class="prettyprint linenums">cd $HOME/cubietruck-build/cubietruck-sdcard
export card=/dev/sdb
export p=""
sudo umount ${card}${p}2
sudo mount ${card}${p}2 /mnt
sudo rm -rv /mnt/*
sudo cp -rv binary/* /mnt
sudo umount ${card}${p}2</xmp>
## 测试
插入sd卡，连接TTL，串口打印信息。
