---
layout: posts
title: "kconfig-language"
---

### 1.Kconfig语法
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
menu/endmenu     生成一个菜单，以endmenu结束
mainmenu         只能出现在配置层次结构的顶部（且只能出现一次）
menuconfig       定义一个配置选项，在这个选项下面还有一个子菜单
config           定义一个配置选项
choice/endchoice 定义一组选择项，用户从一组选项中选择一个选项
source           调用子目录下的Kconfig，生成一个子菜单
comment          在配置选项列表中创建一个注释，注释文本会显示，不能被用户选中
</xmp>
### 2.菜单项属性
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
bool     y/n: 
tristate y/n/M: M表示编译成模块
string
hex
int
</xmp>
### 3.选项之间依赖关系
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
depend on:某选项依赖于另外一个选项生成
select   :反向依赖关系，该选项选中时，同时选中select后面定义的那一项
</xmp>
### 4.默认值:default（默认y/n/m等值）
### 5.输入提示:prompt
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
下面的两个例子是等价的:
   bool "Networking support"
和
   bool
   prompt "Networking support"
</xmp>
### 6.帮助信息:help
### 7.if/else
<xmp style="white-space: pre-wrap; word-wrap: break-word; font-size: 14px;">
1.
  if USB_SUPPORT
  ......
  endif # USB_SUPPORT

2.
  bool "foo" if BAR
  default y if BAR
</xmp>