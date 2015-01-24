---
layout: posts
title: "linux命令行"
---
# {{ page.title }}
## linux命令行文件操作通配符
<xmp class="my_xmp_class">与正则表达式的主要区别：
1.[]、[!]
2.*并不表示*前面字符的重复，而是包括0长度的任意字符串</xmp>
## 虚拟终端与tty
<xmp class="my_xmp_class">Ctrl+Alt+t：打开虚拟终端
Ctrl+Alt+(F1、F2、、F6)：进入tty界面</xmp>
## 环境变量
<xmp class="my_xmp_class">/etc/profile    系统环境变量
bash_profile    用户环境变量
/etc/bashrc     shell变量
.bashrc         用户环境变量
su user         切换用户，加载配置文件.bashrc
su - user       切换用户，加载配置文件/etc/profile ，加载bash_profile</xmp>
## 解压缩
<xmp class="my_xmp_class">tar -[cx]|[zj]vf tar-file [-C] dir</xmp>
## 操作系统名称
<xmp class="my_xmp_class">uname</xmp>
## 文件内容搜索
<xmp class="my_xmp_class">在文件中查找存在指定字符创的行
grep [options] 'strings' file</xmp>
### 1.普通字符：
<xmp class="my_xmp_class">字母、数字、汉字、下划线、以及后边章节中没有特殊定义的标点符号</xmp>
### 2.转义字符：\
<xmp class="my_xmp_class">一些不便书写的字符，采用在前面加 "\" 的方法。这些字符其实我们都已经熟知了。
\r, \n  回车换行符
\t      制表符
\\      \字符本身</xmp>
### 3.字符集合\
<xmp class="my_xmp_class">正则表达式中的一些表示方法，可以匹配 '多种字符' 其中的任意一个字符。
\d      任意一个数字，0~9 中的任意一个
\w      任意一个字母或数字或下划线，也就是 A~Z,a~z,0~9,_ 中任意一个
\s      包括空格、制表符、换页符等空白字符的其中任意一个
\.      小数点可以匹配除了换行符（\n）以外的任意一个字符</xmp>
### 4.自定义字符集合[][^]
<xmp class="my_xmp_class">
[ab5@]      匹配 "a" 或 "b" 或 "5" 或 "@" 
[^abc]      匹配 "a","b","c" 之外的任意一个字符
[f-k]       匹配 "f"~"k" 之间的任意一个字母
[^A-F0-3]   匹配 "A"~"F","0"~"3" 之外的任意一个字符</xmp>
### 5.重复次数{}，前面的单个字符或者()内的分组
<xmp class="my_xmp_class">
{n}         表达式重复n次
{m,n}       表达式至少重复m次，最多重复n次
{m,}        表达式至少重复m次
?           匹配表达式0次或者1次
+           表达式至少出现1次
*           表达式不出现或出现任意次</xmp>
### 6.选择与分组
<xmp class="my_xmp_class">
|       左右两边表达式之间 "或" 关系，匹配左边或者右边
()      (1). 在被修饰匹配次数的时候，括号中的表达式可以作为整体被修饰
        (2). 取匹配结果的时候，括号中的表达式匹配到的内容可以被单独得到</xmp>
## 文件权限
<xmp class="my_xmp_class">R          读         数值表示为4
W          写         数值表示为2
X          可执行     数值表示为1
sudo chmod [u所属用户  g所属组  o其他用户  a所有用户]  [+增加权限  -减少权限]  [r  w  x]   目录名 

例如：有一个文件filename，权限为“-rw-r----x” ,将权限值改为"-rwxrw-r-x"，用数值表示为765
sudo chmod u+x g+w o+r  filename
上面的例子可以用数值表示
sudo chmod 765 filename</xmp>
## touch 新建档案
## vim
<xmp class="my_xmp_class">简化为两种模式
命令模式--》编辑模式：i
编辑模式--》命令模式：Esc
命令模式下：
:q              退出
:q!             强制退出
:wq             保存并退出
:set number     显示行号
:set nonumber   隐藏行号</xmp>
## Ubuntu软件管理
<xmp class="my_xmp_class">sudo add-apt-repository ppa:ubuntu-wine/ppa
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install wine
sudo apt-get remove wine</xmp>