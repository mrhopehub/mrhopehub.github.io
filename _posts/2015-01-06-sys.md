---
layout: posts
title: "/sys目录下各个子目录的具体说明"
---
# {{ page.title }}
<table width="657" cellpadding="4" cellspacing="0">
	
	
	
	<tbody>
		<tr valign="TOP">
			<td style="border-top:1px solid #000000;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0.1cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys</span></span></span>下的子目录
				</p>
			</td>
			<td style="border:1px solid #000000;padding:0.1cm;" width="505">
				<p align="CENTER">
					内容
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/devices</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录下是全局设备结构体系，包含所有被发现的注册在各种总线上的各种物理设备。一般来说，所有的物理设备都按其在总线上的拓扑结构来显示，但有两个例外，即<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">platform
			devices</span></span></span>和<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">system
			devices</span></span></span>。<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">platform
			devices</span></span></span>一般是挂在芯片内部的高速或者低速总线上的各种控制器和外设，它们能被<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">CPU</span></span></span>直接寻址；<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">system
			devices</span></span></span>不是外设，而是芯片内部的核心结构，比如<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">CPU</span></span></span>，<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">timer</span></span></span>等，它们一般没有相关的驱动，但是会有一些体系结构相关的代码来配置它们。
				</p>
				<p>
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(/sys/devices</span></span></span>是内核对系统中所有设备的分层次表达模型，也是<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys</span></span></span>文件系统管理设备的最重要的目录结构<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">)</span></span></span>
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/dev</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录下维护一个按照字符设备和块设备的主次号码<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(major:minor)</span></span></span>链接到真是设备<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(/sys/devices)</span></span></span>的符号链接文件。
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/class</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录下包含所有注册在<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">kernel</span></span></span>里面的设备类型，这是按照设备功能分类的设备模型，每个设备类型表达具有一种功能的设备。每个设备类型子目录下都是这种哦哦那个设备类型的各种具体设备的符号链接，这些链接指向<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/devices/name</span></span></span>下的具体设备。设备类型和设备并没有一一对应的关系，一个物理设备可能具备多种设备类型；一个设备类型只表达具有一种功能的设备，比如：系统所有输入设备都会出现在<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/class/input</span></span></span>之下，而不论它们是以何种总线连接到系统的。<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(/sys/class</span></span></span>也是构成<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">linux</span></span></span>统一设备模型的一部分<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">)</span></span></span>
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/block</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录下的所有子目录代表着系统中当前被发现的所有块设备。按照功能来说防止在<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/class</span></span></span>下会更合适，但由于历史遗留因素而一直存在于<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/block</span></span></span>，但从<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">linux2.6.22</span></span></span>内核开始这部分就已经标记为过去时，只有打开了<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">CONFIG_SYSFS_DEPRECATED</span></span></span>配置编译才会有这个目录存在，并且其中的内容在从<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">linux2.6.26</span></span></span>版本开始已经正式移到了<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/class/block</span></span></span>，旧的接口<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/block</span></span></span>为了向后兼容而保留存在，但其中的内容已经变为了指向它们在<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/devices/</span></span></span>中真实设备的符号链接文件。
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/bus</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录下的每个子目录都是<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">kernel</span></span></span>支持并且已经注册了的总线类型。这是内核设备按照总线类型分层放置的目录结构，<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/devices</span></span></span>中的所有设备都是连接于某种总线之下的，<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">bus</span></span></span>子目录下的每种具体总线之下可以找到每个具体设备的符号链接，
				</p>
				<p>
					一般来说每个子目录<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(</span></span></span>总线类型<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">)</span></span></span>下包含两个子目录，一个是<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">devices</span></span></span>，另一个是<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">drivers</span></span></span>；其中<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">devices</span></span></span>下是这个总线类型下的所有设备，这些设备都是符号链接，它们分别指向真正的设备<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(/sys/devices/name/</span></span></span>下<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">)</span></span></span>；而<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">drivers</span></span></span>下是所有注册在这个总线上的驱动，每个<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">driver</span></span></span>子目录下
			是一些可以观察和修改的<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">driver</span></span></span>参数。
				</p>
				<p>
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(</span></span></span>它也是构成<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">linux</span></span></span>统一设备模型的一部分<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">)</span></span></span>
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/fs</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					按照设计，该目录使用来描述系统中所有的文件系统，包括文件系统本身和按照文件系统分类存放的已挂载点。
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/kernel</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					这个目录下存放的是内核中所有可调整的参数
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/firmware</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录下包含对固件对象<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(firmware
			object)</span></span></span>和属性进行操作和观察的接口，即这里是系统加载固件机制的对用户空间的接口<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">.(</span></span></span>关于固件有专用于固件加载的一套<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">API)</span></span></span>
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/hypervisor</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录是与虚拟化<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">Xen</span></span></span>相关的装置。<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(Xen</span></span></span>是一个开放源代码的虚拟机监视器<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">)</span></span></span>
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/module</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录下有系统中所有的模块信息，不论这些模块是以内联<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(inlined)</span></span></span>方式编译到内核映像文件中还是编译为外模块<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">(.ko</span></span></span>文件<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">)</span></span></span>，都可能出现在<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/module</span></span></span>中。即<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">module</span></span></span>目录下包含了所有的被载入<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">kernel</span></span></span>的模块。
				</p>
			</td>
		</tr>
		<tr valign="TOP">
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:none;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0cm;" width="134">
				<p align="CENTER">
					<br>
				</p>
				<p align="CENTER">
					<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/sys/power</span></span></span>
				</p>
			</td>
			<td style="border-top:none;border-bottom:1px solid #000000;border-left:1px solid #000000;border-right:1px solid #000000;padding-top:0cm;padding-bottom:0.1cm;padding-left:0.1cm;padding-right:0.1cm;" width="505">
				<p>
					该目录是系统中的电源选项，对正在使用的<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">power</span></span></span>子系统的描述。这个目录下有几个属性文件可以用于控制整个机器的电源状态，如可以向其中写入控制命令让机器关机<span style="font-family:'Liberation Serif, serif';"><span style="font-size:small;"><span lang="en-US">/</span></span></span>重启等等。
				</p>
			</td>
		</tr>
	</tbody>
</table>