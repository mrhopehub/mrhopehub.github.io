---
layout: posts
title: "U盘启动整理"
---

# U盘启动
大学时开始倒腾U盘启动，到现在感觉U盘启动也有几个阶段了。

1. DOS启动U盘
2. winPE+ultraISO制作
3. 多启动版本（dos、winPE、Linux），主要是grub的支持
4. 专业做启动的，比如老毛桃、大白菜，做的已经非常好了，这些的技术支持都是FbinstTool，貌似是国外某论坛的大神制作的，超级牛逼。好处不用多说，采用比隐藏分区还隐藏的未使用分区，所以只要不重建分区，即使格式U盘也不影响启动。

现在主要说一下在第四代的基础上，如何使用FbinstTool做一些工作。本文就以<font color="red">老毛桃+PartedMagic的制作为实例。</font>

### 准备工作
软件：

1. 老毛桃U盘启动制作工具，建议20120501这个版本，笔者用的更高的几个版本在恢复系统后，系统都会生成一个绿色IE浏览器（原来备份的时候没有这个绿色IE浏览器）。
2. FbinstTool,最新版本貌似是1.606.2012.1221
3. PartedMagic,大家可能对这个PartedMagic比较陌生，笔者在备份Linux的时候发现的U盘可启动系统（发现的过程很艰辛），GHOST对Linux系统的ext分区不支持，又想像GHOST一样备份Linux，网上找到clonezilla这个系统可以和GHOST媲美。clonezilla本身虽然是一个系统，但是PartedMagic把它作为一个工具做了更高的集成。PartedMagic这个东西好像是收费的。笔者当时在Google上搜到免费下载的，这个[2013.08.01版本](http://www.majorgeeks.com/mg/getmirror/parted_magic,1.html)。

### 开始制作

1. 老毛桃制作，建议分配700M空间或者更多，因为还要加入PartedMagic。如图<br>![老毛桃](/images/U盘启动/老毛桃.jpg)
2. 运行FbinstTool工具，选择正确的U盘
3. pmagic_2013_08_01.iso中解压出pmagic目录，用FbinstTool把该目录添加到U盘UD分区的BOOT目录下。如图<br>![老毛桃](/images/U盘启动/大神工具.jpg)
4. 添加grub启动项。<font color="red">注意添加之后不要忘记保存。</font><br><br>
title 【12】 PartedMagic<br>
kernel (ud)/BOOT/pmagic/bzImage<br>
initrd (ud)/BOOT/pmagic/initrd.img<br>

### 备份与恢复U盘启动,或者说更简单的制作方法（注意备份U盘数据）
>备份

>1. 启动FbinstTool工具，数据管理-->备份ud到fba文件

>恢复

>1. 启动FbinstTool工具，数据管理-->导入fba文件