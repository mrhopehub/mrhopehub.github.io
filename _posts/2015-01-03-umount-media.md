---
layout: posts
title: "linux卸载命令"
---
# {{ page.title }}
如下两个脚本
<xmp class="prettyprint linenums">#! /bin/sh
#脚本1
export card=/dev/sdb
export p=""

sudo umount ${card}${p}1
sudo mount ${card}${p}1 /mnt
cd /mnt
sudo umount ${card}${p}1</xmp>
<xmp class="prettyprint linenums">#! /bin/sh
#脚本2
mkdir tmp
cd tmp
rm -rv ../tmp</xmp>
脚本1会提示以下内容
<xmp class="my_xmp_class">umount: /mnt: device is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))</xmp>
而脚本2不会提示错误。是因为cd在某个挂载时不能umount吗(脚本1去掉最后一行，再次运行脚本1结束，然后在终端中运行最后一句则没有任何提示。)？而cd在某个目录时就可以删除该目录。