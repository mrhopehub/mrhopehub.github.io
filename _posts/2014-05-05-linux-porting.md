---
layout: posts
title: "关于嵌入式系统的移植"
---

# 关于嵌入式系统的移植
### 简介
无论是bootloader的移植还是linux的移植，都要明白那些工作已经做好了、那些工作还没有做。主要分为两部分：<font color="red">架构部分、板级部分。</font>
##### <font color="blue">架构部分</font>
这部分工作一般由架构厂商来做，那uboot来说，arch/arm/cpu/armv7/start.S文件、arch/arm/cpu/armv7/lowlevel_init.S文件。已经完成了arm核基本的部分。
##### <font color="blue">板级部分</font>
这里要区分什么是板级部分，辩证的说法就是除了arm核部分。但是要注意几点：

* arm核部分已经做了什么工作，比如中断、cache、处理器模式、堆栈设置
* arm核部分留给板级部分的接口是什么，比如void s_init(void)
* 板级部分需要完成的基本工作，比如go setup pll, mux, memory

##### <font color="blue">总结</font>
所以要弄懂bootloader的流程，熟悉流程中：

* 每一步的作用是什么
* <font color="red">每一步属于架构部分还是板级部分或者是上层部分（注意这里说的上层部分是从这个系统的分层角度来说的，比如内核的调度器初始化sched_init()）</font>
* 给板级部分留出来的接口是什么。