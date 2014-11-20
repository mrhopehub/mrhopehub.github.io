---
layout: posts
title: "bios、dts、uboot"
---

# {{ page.title }}
转载：http://course1.scetc.net/wjzz/dispArticle.Asp?ID=331
<xmp class="my_xmp_class">
win98启动过程详解
 (一)、BIOS的启动过程 
只要一打开计算机的电源开关，一个叫Bootstrap(引导程序)的小软件就要发挥作用，它常驻在BIOS ROM的地址FFFFOH处,当ROM加载程序加载它后，它就完成下面的几项工作。
1、加电自检
POST（Post On Self Text，加电自检子程序）测试系统的完整性，如果系统通过测试，计算机扬声器发出一短促的鸣叫声（是否有鸣叫声取决于BIOS的厂家，这里以广泛使用的Award的BIOS为例），如果系统出现故障而未通过测试，根据故障的不同扬声器将发出不同的鸣叫声，因为各个厂商对鸣叫声的定义不同，要了解各个鸣叫声的意义需查看BIOS或者主板厂商的用户手册。某些BIOS在检测出系统故障时会暂停并且在显示器上显示出相关的错误信息（如键盘错误等）。在POST过程还要测试内存的完整性。
2、检测即插即用设备
3、查找引导盘
引导程序接着检测BIOS中的设置以找到第一个可引导的驱动器（一般为A盘或C盘），如果在检测完BIOS中指定的所有可引导器仍未发现引导驱动器，引导程序暂停启动过程并显示一个错误信息：找不到启动盘。
（二）、Dos的启动过程
操作系统加载程序从ROM加载程序得到控制权后就开始DOS的启动，其步骤如下。
1、加载IO.SYS
操作系统加载程序从引导驱动器上读取主引导记录MBR（Master Boot Record）并将控制权叫给MBR，MBR读取分区表（在MBR的尾部）并找到引导分区的位置，MBR将控制权叫给引导分区的引导扇区（引导扇区包含磁盘引导程序和磁盘特性表）上的磁盘引导程序，检测BIOS参数块（BPB，BIOS Parameter Block）以找到操作系统引导文件所在的根目录，将操作系统引导文件IO.SYS从根目录拷贝进内存，IO.SYS实际上是一个可执行文件并且只能位于引导分区的第一磁道上。
2、加载FAT和MSDOS.SYS
3、处理CONFIG.SYS和AUTOEXEC.BAT
如果CONFIG.SYS文件不存在，IO.SYS从MSDOS.SYS的“WinBootDir=”获得Ifshlp.sys、Himem.sys和Setver.exe这三个文件的位置，然后自动加载这三个必需的驱动程序。如果MSDOS.SYS中有BootGUI=0这个选项，IO.SYS将控制权交给命令行解释器COMMAND.COM（或者叫给CONFIG.SYS中由命令“SHELL=”指定的命令行解释器），然后COMMAND.COM将控制权叫给计算机用户，也就是等待用户输入DOS命令，至此DOS的启动过程完成。
（三）、Windows的启动过程
在DOS启动过程的最后一步，如果MSDOS.SYS中是BootGUI=1而不是BootGUI=0这个选项，IO.SYS将控制权将交给Windows加载程序以继续加载Windows，Windows的启动过程真正开始。
1、显示“Starting Windows...”
屏幕显示“Starting Windows 9x...”这个提示信息，在这个信息显示的过程中：
MSDOS.SYS中的BootDelay=n（n为整数）选项可以控制该信息的显示延长时间，若MSDOS.SYS没有该选项，默认该信息显示3秒。
若MSDOS.SYS中有BootKeys=1，按住Ctrl或F8键则显示Windows启动菜单。若有BootMenu=1，不按住Ctrl或F8键也会自动显示Windows启动菜单。
MSDOS.SYS中若有BootMenuDelay=n的选项，可以指定Windows启动菜单显示的延长时间，默认是30秒。
MSDOS.SYS中若有BootMenuDefault=n，可以指定Windows启动菜单上的启动项，默认是1，即以正常模式启动Windows。
如果Windows上一次没有正常关闭，而且在MSDOS.SYS中没有AutoScan=0选项，磁盘扫描程序Scandisk将询问或者自动扫描硬盘，默认该选项是AutoScan=1，既自动扫描。
不管MSDOS.SYS中是否有以上选项，只要Windows上一次的启动或关机过程没有正常完成，Windows的启动菜单会自动出现，而且默认启动项是安全模式（Windows Safe Mode），这个自动出现的启动菜单其显示延时是30秒。
在Windows的启动过程中，将保留所有的UMB（Upper Memory Block，上位内存）使用。
2、检测Windows的启动画面
如果MSDOS.SYS中有logo=1选项或者没有该选项，IO.SYS加载并显示其内部默认的Windows启动画面（即蓝天白云画面）。用户可自定义一个LOGO.SYS文件（实际上是分辨率为320╳400、颜色深度为256色的BMP图形）并把它放在根目录下一取代该画面，这样Windows的启动画面就变成了用户的自定义画面。可能有些计算机用户这样做后发现显示的仍然是蓝天白云画面，这种情况发生在OEM版的Windows中，原因是微软为这些OEM厂商修改了LOGO.SYS必须放在特定的目录中，例如C:\WINDOWS或者别的目录中，不同的OEM厂家可能有所不同。
在MSDOS.SYS设置logo=0则不显示Windows的启动画面。
3、检测DRVSPACE.INI和DBLSPACE.INI文件
如果存在DRVSPACE.INI和DBLSPACE.INI文件，并且在MSDOS.SYS中没有指定DblSpace=0、Drvspace=0，DRVSPACE.BIN和DBLSPACE.BIN被加载。
4、检测注册表
IO.SYS打开注册表文件SYSTEM.DAT并调用其它工具检测数据的有效性，如果文件SYSTEM.DAT不存在，则自动从备份文件中恢复该文件，如果SYSTEM.DAT被恢复，USER.DAT也被自动恢复。Windows98中备份文件被压缩在目录C:\WINDOWS\SYSBACKUP\下的RB00n.CAB中，n的值为0—5。
5、检测DBLBUFF.SYS
如果在MSDOS.SYS中有DoubleBuffer=1，或者注册表中有键值HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\WinBoot\DoubleBuffer，则加载DBLBUFF.SYS。即使以上两个条件都没有满足，但是Windows探测到双缓冲（Double Buffer）是必须的，也会自动加载双缓冲。
6、加载WIN.COM
配置文件加载完成后即运行WIN.COM，WIN.COM是Windows的加载器（Windows Loader），由它继续Windows的启动工作。
7、加载Vxd文件
Vxd其全称为Virtual Device Driver，即虚拟设备驱动程序。WIN.COM首先处理VMM32.VXD。
实模式虚拟设备驱动程序加载程序检测是否所需的VxD文件已成功加载，如果没有，它再一次尝试加载。一旦实模式虚拟设备驱动程序加载成功，设备初始化开始。任何需要在实模式下初始化的VxD文件此时开始初始化。VMM32将计算机处理器从实模式切换到保护模式，VxD第三阶段的初始化过程开始。在这个阶段VxD设备驱动程序根据InitDevice指定的次序而不是根据VxD被加载进入到内存中的次序进行初始化，这些VxD文件初始化次序如下。
a.SYS_CRITICAL_INIT(SYSCRITINIT,系统关键初始化）
在这个阶段为了让VxD文件有足够多的时间准备设备初始化而不被系统中断，所有的系统中断都被关闭，所有的文件输入/输出（I/O）也被关闭，因此所有的VxD文件被加载的过程不被Windows启动记录文件Boot.txt记录，直到该初始化过程完成之后，所有VxD文件被加载的过程才被记录到文件Bootlog.txt中。
b、SYS_DEVICE_INIT(DEVICEINIT,系统设备初始化）
在这个阶段大量的VxD进行初始化，文件的输入/输出也被允许，因此每一个VxD文件的初始化都被记录，但Ifsmgr的设备初始化例外。Ifsmgr的作用是控制实模式文件系统，在Ifsmgr的设备初始化过程中磁盘输入/输出不被允许，直到其初始化完成后磁盘输入/输出才被允许进行。由于这个原因，Ifsmgr的初始化过程也没有被记录，因此从表面上看，好像它在设备初始化阶段并没有出现。
c、SYS_INIT_COMPLETE(INITCOMPLETE,系统初始化完成）
通过这几个阶段的VxD此时一般可以正常工作了，而那些通过a、b两阶段而没有通过c阶段的VxD将被从内存中清除。
8、加载GUI程序
在所有的静态VxD和WINSTART.BAT被加载后，Windows的GUI（Graphical User Interface，图形用户界面）被加载，这些GUI程序是Krnl32.dll、Gdi.exe、User.exe和Explorer.exe，其中Explorer.exe是Windows默认的Shell，可以改为使用别的应用程序来代替。
9、Windows注册和网络注册
接下来是加载网络环境设置，即加载注册表键值HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce、Windows logon和network,此时出现询问Windows注册和网络注册密码的对话框。Windows加载网络环境参数时提示用户输入网络注册密码，如果用户是在单机上而不是在网络上使用计算机，并且已经关闭了密码输入提示功能，Windows将用以前提供的默认用户名实现自动注册网络，Windows要完成自动注册功能必须满足：
（1）以前至少输入过一次有效的用户名；
（2）上次输入的用户名没有被清除；
（3）没有设置必须使用密码。
在单机用户系统上按ESC或者选择取消，Windows将会使用默认的桌面设置继续启动，但Widows下一次启动时会再次要求输入用户名。如果网络注册验证中设置不完全正确且用户输入了一个新的用户名，Windows将根据控制面板中的网络用户设置参数替这个用户创造一个环境设置参数。
10、注册表主键加载
Windows中有几个自动运行的项目，它们按以下的次序加载：
(1)HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunservicesOnce,
(2)HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Runservices,
(3)Windows的注册提示，
(4)HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce,
(5)HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run,
(6)HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run,
(7)启动组，
(8)HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce。
上面各项(1)、(2)和(3)可以同时加载，只有(4)的地位比较特殊，它必须等待(1)、(2)和(3)都完成加载之后才开始加载，而在它之后的(5)、(6)、(7)和(8)也都可以同时加载，但它们都必须等待(4)中所有的项目完成加载之后才开始加载。RunservicesOnce中的键值在执行一次之后被自动地从注册表中清除。
经过了以上的所有步骤，Windows也就完成了其启动的全过程。
 
使用DEBUG破解CMOS密码
 
　　经常由于忘记或不知口令而无法进入BIOS设置状态或无法进入系统，这时可采用下面的方法破解。应该注意的是，COMS密码分两种，一种是Setup密码，另一种是System密码（它们是通过BIOS设置的，具体请参考BIOS设置篇中的芯片部分及密码设置部分)
　　如果选择System，那么每次开机启动时都会提示您输入密码，如果密码不对，那么就无法使用计算机了，此密码的设置目的在于禁止外来者使用计算机；如果选择Setup，那么仅在进入CMOS设置时才提示您输入密码，此密码设置的目的在于禁止未授权用户设置BIOS。我们可根据不同的目的进行设置，一般来讲，设置了SYSTME密码，那么安全性更高些，但同时如果忘记密码，其破解也就更复杂些，而设置了Setup密码则反之。下面就列出常用的CMOS密码破解方法：
　　DEBUG法 用DEBUG(DOS自带的一个程序)向端口70h和71h发送一个数据，可以清除口令设置，具体操作如下： 
　　C:\>DEBUG 
　　―O 70 10 
　　―O 71 01 
　　―Q 
　　另外可以把上述操作用DEBUG写成一个程序放在一个文件（如DELCMOS.COM）中，具体操作如下： 
　　C:\>DEBUG
　　―A 100 
　　XXXX:0100 MOV DX,70 
　　XXXX:0103 MOV AL,10 
　　XXXX:0105 OUT DX,AL 
　　XXXX:0106 MOV DX,71 
　　XXXX:0109 MOV AL,01 
　　XXXX:010B OUT DX,AL 
　　XXXX:010C 
　　―R CX 
　　CX 0000 
　　: 0C 
　　―N DELCMOS.COM 
　　―W 
　　Writing 000C bytes 
　　―Q 
　　以后，运行DELCMOS.COM就能清除口令设置了。
 
计算机启动过程详解  
　　打开电源启动机器几乎是电脑爱好者每天必做的事情，面对屏幕上出现的一幅幅启动画面，我们一点儿也不会感到陌生，但是，计算机在显示这些启动画面时都做了些什么工作呢？相信有的朋友还不是很清楚，本文就来介绍一下从打开电源到出现Windows 9x的蓝天白云时，计算机到底都干了些什么事情。
　　首先让我们来了解一些基本概念。第一个是大家非常熟悉的BIOS(基本输入输出系统)，BIOS是直接与硬件打交道的底层代码，它为操作系统提供了控制硬件设备的基本功能。BIOS包括有系统BIOS(即常说的主板BIOS)、显卡BIOS和其它设备(例如IDE控制器、SCSI卡或网卡等)的BIOS，其中系统BIOS是本文要讨论的主角，因为计算机的启动过程正是在它的控制下进行的。BIOS一般被存放在ROM(只读存储芯片)之中，即使在关机或掉电以后，这些代码也不会消失。
　　第二个基本概念是内存的地址，我们的机器中一般安装有32MB、64MB或128MB内存，这些内存的每一个字节都被赋予了一个地址，以便CPU访问内存。32MB的地址范围用十六进制数表示就是0～1FFFFFFH，其中0～FFFFFH的低端1MB内存非常特殊，因为最初的8086处理器能够访问的内存最大只有1MB，这1MB的低端640KB被称为基本内存，而A0000H～BFFFFH要保留给显示卡的显存使用，C0000H～FFFFFH则被保留给BIOS使用，其中系统BIOS一般占用了最后的64KB或更多一点的空间，显卡BIOS一般在C0000H～C7FFFH处，IDE控制器的BIOS在C8000H～CBFFFH处。
　　好了，下面我们就来仔细看看计算机的启动过程吧。
　　第一步：当我们按下电源开关时，电源就开始向主板和其它设备供电，此时电压还不太稳定，主板上的控制芯片组会向CPU发出并保持一个RESET(重置)信号，让CPU内部自动恢复到初始状态，但CPU在此刻不会马上执行指令。当芯片组检测到电源已经开始稳定供电了(当然从不稳定到稳定的过程只是一瞬间的事情)，它便撤去RESET信号(如果是手工按下计算机面板上的Reset按钮来重启机器，那么松开该按钮时芯片组就会撤去RESET信号)，CPU马上就从地址FFFF0H处开始执行指令，从前面的介绍可知，这个地址实际上在系统BIOS的地址范围内，无论是Award BIOS还是AMI BIOS，放在这里的只是一条跳转指令，跳到系统BIOS中真正的启动代码处。
　　第二步：系统BIOS的启动代码首先要做的事情就是进行POST(Power-On Self Test，加电后自检)，POST的主要任务是检测系统中一些关键设备是否存在和能否正常工作，例如内存和显卡等设备。由于POST是最早进行的检测过程，此时显卡还没有初始化，如果系统BIOS在进行POST的过程中发现了一些致命错误，例如没有找到内存或者内存有问题(此时只会检查640K常规内存)，那么系统BIOS就会直接控制喇叭发声来报告错误，声音的长短和次数代表了错误的类型。在正常情况下，POST过程进行得非常快，我们几乎无法感觉到它的存在，POST结束之后就会调用其它代码来进行更完整的硬件检测。
　　第三步：接下来系统BIOS将查找显卡的BIOS，前面说过，存放显卡BIOS的ROM芯片的起始地址通常设在C0000H处，系统BIOS在这个地方找到显卡BIOS之后就调用它的初始化代码，由显卡BIOS来初始化显卡，此时多数显卡都会在屏幕上显示出一些初始化信息，介绍生产厂商、图形芯片类型等内容，不过这个画面几乎是一闪而过。系统BIOS接着会查找其它设备的BIOS程序，找到之后同样要调用这些BIOS内部的初始化代码来初始化相关的设备。
　　第四步：查找完所有其它设备的BIOS之后，系统BIOS将显示出它自己的启动画面，其中包括有系统BIOS的类型、序列号和版本号等内容。
　　第五步：接着系统BIOS将检测和显示CPU的类型和工作频率，然后开始测试所有的RAM，并同时在屏幕上显示内存测试的进度，我们可以在CMOS设置中自行决定使用简单耗时少或者详细耗时多的测试方式。
　　第六步：内存测试通过之后，系统BIOS将开始检测系统中安装的一些标准硬件设备，包括硬盘、CD-ROM、串口、并口、软驱等设备，另外绝大多数较新版本的系统BIOS在这一过程中还要自动检测和设置内存的定时参数、硬盘参数和访问模式等。
　　第七步：标准设备检测完毕后，系统BIOS内部的支持即插即用的代码将开始检测和配置系统中安装的即插即用设备，每找到一个设备之后，系统BIOS都会在屏幕上显示出设备的名称和型号等信息，同时为该设备分配中断、DMA通道和I/O端口等资源。
　　第八步：到这一步为止，所有硬件都已经检测配置完毕了，多数系统BIOS会重新清屏并在屏幕上方显示出一个表格，其中概略地列出了系统中安装的各种标准硬件设备，以及它们使用的资源和一些相关工作参数。
　　第九步：接下来系统BIOS将更新ESCD(Extended System Configuration Data，扩展系统配置数据)。ESCD是系统BIOS用来与操作系统交换硬件配置信息的一种手段，这些数据被存放在CMOS(一小块特殊的RAM，由主板上的电池来供电)之中。通常ESCD数据只在系统硬件配置发生改变后才会更新，所以不是每次启动机器时我们都能够看到“Update ESCD…Success”这样的信息，不过，某些主板的系统BIOS在保存ESCD数据时使用了与Windows 9x不相同的数据格式，于是Windows 9x在它自己的启动过程中会把ESCD数据修改成自己的格式，但在下一次启动机器时，即使硬件配置没有发生改变，系统BIOS也会把ESCD的数据格式改回来，如此循环，将会导致在每次启动机器时，系统BIOS都要更新一遍ESCD，这就是为什么有些机器在每次启动时都会显示出相关信息的原因。
　　第十步：ESCD更新完毕后，系统BIOS的启动代码将进行它的最后一项工作，即根据用户指定的启动顺序从软盘、硬盘或光驱启动。以从C盘启动为例，系统BIOS将读取并执行硬盘上的主引导记录，主引导记录接着从分区表中找到第一个活动分区，然后读取并执行这个活动分区的分区引导记录，而分区引导记录将负责读取并执行IO.SYS，这是DOS和Windows 9x最基本的系统文件。Windows 9x的IO.SYS首先要初始化一些重要的系统数据，然后就显示出我们熟悉的蓝天白云，在这幅画面之下，Windows将继续进行DOS部分和GUI(图形用户界面)部分的引导和初始化工作。
　　如果系统之中安装有引导多种操作系统的工具软件，通常主引导记录将被替换成该软件的引导代码，这些代码将允许用户选择一种操作系统，然后读取并执行该操作系统的基本引导代码(DOS和Windows的基本引导代码就是分区引导记录)。
　　上面介绍的便是计算机在打开电源开关(或按Reset键)进行冷启动时所要完成的各种初始化工作，如果我们在DOS下按Ctrl＋Alt＋Del组合键(或从Windows中选择重新启动计算机)来进行热启动，那么POST过程将被跳过去，直接从第三步开始，另外第五步的检测CPU和内存测试也不会再进行。我们可以看到，无论是冷启动还是热启动，系统BIOS都一次又一次地重复进行着这些我们平时并不太注意的事情，然而正是这些单调的硬件检测步骤为我们能够正常使用电脑提供了基础。
 
主板BIOS基础知识
    BIOS英文全称是Basic Input/Output System，完整地说应该是ROM－BIOS，是只读存储器基本输入／输出系统的简写，它实际上是被固化到计算机中的一组程序，为计算机提供最低级的、最直接的硬件控制。准确地说，BIOS是硬件与软件程序之间的一个“转换器”或者说是接口(虽然它本身也只是一个程序)，负责解决硬件的即时需求，并按软件对硬件的操作要求具体执行。
　　BIOS的功能：
　　从功能上看，BIOS分为三个部分：
　　1.自检及初始化程序；
　　2.硬件中断处理；
　　3.程序服务请求。

    BIOS的种类：
　　由于BIOS直接和系统硬件资源打交道，因此总是针对某一类型的硬件系统，而各种硬件系统又各有不同，所以存在各种不同种类的BIOS，随着硬件技术的发展，同一种BIOS也先后出现了不同的版本，新版本的BIOS比起老版本来说，功能更强。
　　目前市场上主要的BIOS有AMI BIOS和Award BIOS。
　　1.AMI BIOS
　　AMI BIOS是AMI公司出品的BIOS系统软件，最早开发于80年代中期，为多数的286和386计算机系统所采用，因对各种软、硬件的适应性好、硬件工作可靠、系统性能较佳、操作直观方便的优点受到用户的欢迎。
　　90年代，AMI又不断推出新版本的BIOS以适应技术的发展，但在绿色节能型系统开始普及时，AMI似乎显得有些滞后，Award BIOS的市场占有率借此机会大大提高，在这一时期，AMI研制并推出了具有窗口化功能的WIN BIOS，这种BIOS设置程序使用非常方便，而且主窗口的各种标记也比较直观，例如，一只小兔子表示优化的默认设置，而一只小乌龟则表示保守的设置，一个骷髅用来表示反病毒方面的设置，画笔和调色板则表示色彩的设置。
　　AMI WinBIOS已经有多个版本，目前用得较多的有奔腾机主板的Win BIOS，具有即插即用、绿色节能、PCI总线管理等功能。
　　2.Award BIOS
　　Award BIOS是Award Software公司开发的BIOS产品，目前十分流行，许多586主板机都采用Award BIOS，功能比较齐全，对各种操作系统提供良好的支持。Award BIOS也有许多版本，现在用得最多的是4.X版。
    通过BIOS自检音识别硬件状态：
    计算机启动后就通过BIOS对计算机进行自检。自检情况一般通过PC喇叭发出的响铃予以表达。了解这种响铃，对于诊断计算机硬件故障大有裨益。每个品牌的BIOS自检响铃所表达的意义有所不同。其具体意义如下：
　　Award BIOS
　　1短：系统正常启动。恭喜，你的机器没有任何问题。
　　2短：常规错误，请进入CMOS Setup，重新设置不正确的选项。
　　1长1短：RAM或主板出错。换一条内存试试，若还是不行，只好更换主板。
　　1长2短：显示器或显示卡错误。
　　1长3短：键盘控制器错误。检查主板。
　　1长9短：主板Flash RAM或EPROM错误，BIOS损坏。换块Flash RAM试试。
　　不断地响（长声）：内存条未插紧或损坏。重插内存条，若还是不行，只有更换一条内存。
　　不停地响：电源、显示器未和显示卡连接好。检查一下所有的插头。
　　重复短响：电源有问题。
　　无声音无显示：电源有问题。
　　AMI BIOS
　　1短：内存刷新失败。更换内存条。
　　2短：内存ECC较验错误。在CMOS Setup中将内存关于ECC校验的选项设为Disabled就可以解决，不过最根本的解决办法还是更换一条内存。
　　3短：系统基本内存（第1个64kB）检查失败。换内存。
　　4短：系统时钟出错。
　　5短：中央处理器（CPU）错误。
　　6短：键盘控制器错误。
　　7短：系统实模式错误，不能切换到保护模式。
　　8短：显示内存错误。显示内存有问题，更换显卡试试。
　　9短：ROM BIOS检验和错误。
　　1长3短：内存错误。内存损坏，更换即可。
　　1长8短：显示测试错误。显示器数据线没插好或显示卡没插牢。
</xmp>
# <font color="blue">总结</font>
bios作用（个人理解）：

* 初始化内存等操作系统不知道的设备（<font color="red">uboot初始化内存控制器类似</font>）
* 存放板级硬件配置资源提供给操作系统用于硬件探测（多指PCI桥资源，而不关心PCI设备资源，<font color="red">类似ARM嵌入式的DTS</font>）
* 存放个别设备的firmware或者说bios（<font color="red">操作系统不使用的时候需要自己实现</font>）