---
layout: posts
title: "关于linux编译的Kconfig、makefile、.config文件"
---

# {{ page.title }}
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
曾几何时，linux是多么的神秘，多么的高深莫测，区区编译一个内核就搞得摸不到头脑，更别说Kconfig、makefile、.config这些文件。

编译linux内核时make bzImage命令是真正的开始编译内核，说到make就不得不说makefile了，所以编译内核最终是根据makefile文件指示的，而.config、Kconfig文件只是辅助，更具体一点，当用户make menuconfig的时候其实是根据Kconfig文件来生成图形菜单，用户配置完图形菜单就会保存到.config文件中，makefile就可以引用.config文件中的内容（其实是makefile相应用户的配置），所以这里不得不说makefile完全可以不引用.config，也就是无视用户的配置，直接写下makefile文件。那么就有一下几点1、即使用户选择了某个模块，makefile文件没有相应的编译，最终也不会编译的2、即使用户选择了模块，makefile完全可以无视之，直接编译到内核镜像中，而不是编译为模块。所以在内核中增加驱动是就可以考虑，不用修改Kconfig文件，直接修改makefile文件，当然按照惯例是要同时修改两个文件的。

最后要说的是，可见Kconfig、makefile、.config既有联系又相互独立，而且功能有些重复，其实就是系统的复杂性与人性化。整体上的表现就是B模块可以依赖A模块，也可以不依赖A模块，也可以完全可以和A模块相反，但正常的情况是B模块还是依赖A模块而且不是变现的相反性。自己在做系统设计的时候可以参考这一点。

文本方式的复杂性与灵活性，由于都是文本一个make config之类的就有很多选项，如make config、make defconfig、make xxx_config，所以对文本方式的总结是——容易引起歧义，选项较多（这一点正如linux的命令行一样），有时候漏掉一些情况（读者需要自己去判断，如makefile与.config的三种情况），但优点是特别灵活，shell脚本就是这样，比如每次编译过内核之后，然后安装（sudo make install）现在的问题来了，这个安装包括更新grub吗？如果包括了更新grub那么就不用（sudo update-grub），这就是文本方式的问题。这里还要说，程序设计本身就是文本方式，上面对应与一个功能应该由哪个函数实现或者说是耦合的问题
</xmp>