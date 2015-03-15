---
layout: posts
title: "简陋字符驱动"
---
# {{ page.title }}
## ubuntu版本号13.10，接着上一篇hello module。
整个寒假在家没怎么看linux内核方面的东西，忘了很多东西。参考linux/drivers/char/scx200_gpio.c写了一个及其朴素的字符设备驱动
## 1.切换到root账户
<xmp class="prettyprint linenums">sudo su</xmp>
## 2.创建工程目录
<xmp class="prettyprint linenums">mkdir char-device
cd char-device</xmp>
## 3.源文件、Makefile文件
char-device.c
<xmp class="prettyprint linenums">#include <linux/device.h>
#include <linux/fs.h>
#include <linux/module.h>
#include <linux/errno.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/platform_device.h>
#include <asm/uaccess.h>
#include <asm/io.h>

#include <linux/types.h>
#include <linux/cdev.h>
MODULE_AUTHOR("mrhopehub <mrhopehub@163.com>");
MODULE_DESCRIPTION("my first char driver");
MODULE_LICENSE("GPL");

char file_private_data = 'o';
static int major = 0;		/* default to dynamic major */

static int char_device_open(struct inode *inode, struct file *file)
{
	file->private_data = &file_private_data;
	return 0;
}
ssize_t char_device_write(struct file *file, const char __user *data,
		       size_t len, loff_t *ppos)
{
	int i;
	char c;
	char *tmp = file->private_data;
	char private = *tmp;
	for(i=0; i<len; i++)
	{
		get_user(c, data + i);
		if(c == private)printk("第%d个写入的字符,与私有数据相等。\n",i);
		else printk("与私有数据不想等的字符：%c\n",c);
	}
	return len;
}
static const struct file_operations char_device_fileops = {
	.owner   = THIS_MODULE,
	.write   = char_device_write,
	.open    = char_device_open,
};

static struct cdev my_device_cdev;
static int __init hello_device_init(void)
{
	int rc;
	dev_t devid;
	rc = alloc_chrdev_region(&devid, 0, 1, "my-char-device");
	major = MAJOR(devid);
	if(rc < 0)
	{
		printk("设备号分配出错！！\n");
		return -1;
	}
	cdev_init(&my_device_cdev, &char_device_fileops);
	cdev_add(&my_device_cdev, devid, 1);
	printk("\n模块初始化成功！\n");
	return 0;
}
static void __exit hello_device_cleanup(void)
{
	cdev_del(&my_device_cdev);
	unregister_chrdev_region(MKDEV(major, 0), 1);
	printk("\n模块移除成功！\n");
}
module_init(hello_device_init);
module_exit(hello_device_cleanup);
</xmp>
Makefile文件
<xmp class="prettyprint linenums">obj-m += char-device.o
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
<xmp class="prettyprint linenums">insmod char-device.ko</xmp>
## 6.查看设备对应的主设备号
<xmp class="prettyprint linenums">cat /proc/devices</xmp>
## 7./dev目录下创建设备节点
（假设上一步看到的my-char-device的主设备号是250，系统分配，笔者主机上分配的250）
<xmp class="prettyprint linenums">mknod /dev/hello-cdev c 250 0</xmp>
## 8.测试写入函数，既写入hello-cdev文件一些字符
<xmp class="prettyprint linenums">echo oooo > /dev/hello-cdev</xmp>
## 9.查看结果
<xmp class="prettyprint linenums">cat /var/log/kern.log</xmp>
## 10.删除设备节点、卸载模块
<xmp class="prettyprint linenums">rm /dev/hello-cdev
rmmod char-device</xmp>