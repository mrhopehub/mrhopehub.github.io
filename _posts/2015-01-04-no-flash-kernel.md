---
layout: posts
title: "去掉内核中的NAND flash驱动"
---
# {{ page.title }}
上次笔者制作的可启动SD卡虽然成功，但是重新从NAND启动启动Android时，并不能启动，具体的串口错误忘了记下来，主要是flash分区错误，所以想到制作可启动SD卡时，配置内核时把NAND驱动去掉即可。

## 修改内核配置
<xmp class="prettyprint linenums">export PATH=$HOME/cubietruck-build/cubietruck-uboot/u-boot-sunxi/tools:$PATH
cd $HOME/cubietruck-build/cubietruck-kernel/linux-sunxi
rm -rv .config
cp ../cubie_configs/kernel-configs/3.4/cubietruck_defconfig .config
make ARCH=arm menuconfig</xmp>
<xmp class="my_xmp_class">--->Device Drivers
    --->Block Devices
        --->SUNXI Nandflash Driver
        --->Create old nand device names (nanda-nandz) (NEW)</xmp>
<xmp class="my_xmp_class">--->Enable the block layer
    --->Partition Types
        --->sunxi nand partition table support</xmp>
<xmp style="color: red; font-size: 14px;" class="my_xmp_class">按序将下列选项选择N
1.Create old nand device names (nanda-nandz) (NEW)
2.SUNXI Nandflash Driver
3.sunxi nand partition table support</xmp>
## 编译内核
<xmp class="prettyprint linenums">make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage modules</xmp>
但是会有如下结果
<xmp class="prettyprint linenums">Image Name:   Linux-3.4.103+
Created:      Sun Jan  4 15:18:37 2015
Image Type:   ARM Linux Kernel Image (uncompressed)
Data Size:    4548312 Bytes = 4441.71 kB = 4.34 MB
Load Address: 40008000
Entry Point:  40008000
  Image arch/arm/boot/uImage is ready
  Building modules, stage 2.
  MODPOST 1139 modules
WARNING: modpost: Found 1 section mismatch(es).
To see full details build your kernel with:
'make CONFIG_DEBUG_SECTION_MISMATCH=y'</xmp>
不知道WARNING原因是什么，但是笔者测试过，<xmp style="color: red; font-size: 14px;" class="my_xmp_class">内核第一次编译成功之后，即使不改变config，再次编译也是上面的WARNING。</xmp>
## 测试
如果已经制作好了可启动的SD卡，只需要用这次编译好的内核替换原来的内核即可。
<xmp style="color: red; font-size: 14px;" class="my_xmp_class">用PhoenixSuit安装Android到Cubietruck，确认可以启动。</xmp>
关机之后插入SD卡，Cubietruck从SD卡启动。
<xmp style="color: red; font-size: 14px;" class="my_xmp_class">再关机之后，拔掉SD卡，Cubietruck从NAND启动安卓成功。</xmp>