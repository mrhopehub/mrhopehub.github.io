---
layout: posts
title: "ubuntu下hello world模块"
---
# {{ page.title }}
## ubuntu版本号13.10，接着上一篇内核源码树。
## 1.切换到root账户
<xmp class="prettyprint linenums">sudo su</xmp>
## 2.创建工程目录
<xmp class="prettyprint linenums">mkdir hello-module
cd hello-module</xmp>
## 3.源文件、Makefile文件
hello.c
<xmp class="prettyprint linenums">#include<linux/init.h>
#include<linux/kernel.h>
#include<linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");
int hello_init(void)
{
	printk("hello,module\n");
	return 0;
}
void hello_exit(void)
{
	printk("exit,module\n");
}
module_init(hello_init);
module_exit(hello_exit);
</xmp>
Makefile
<xmp class="prettyprint linenums">obj-m += hello.o
KERNELDIR := /usr/src/linux
PWD := $(shell pwd)  
modules:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules
modules_install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install  
clean:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) clean</xmp>
## 4.编译模块
<xmp class="prettyprint linenums">make</xmp>
## 5.加载模块
<xmp class="prettyprint linenums">insmod ./hello.ko</xmp>
## 6.查看模块列表
<xmp class="prettyprint linenums">lsmod</xmp>
## 7.卸载模块
<xmp class="prettyprint linenums">rmmod ./hello.ko</xmp>
## 8.查看printk信息
在网上找了很多关于printk输出不到控制台的信息，暂时没有解决，但是可以通过log文件查看。
<xmp class="prettyprint linenums">cat /var/log/kern.log</xmp>