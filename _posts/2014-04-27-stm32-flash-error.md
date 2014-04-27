---
layout: posts
title: "stm32:error:Flash Download failed - "Cortex-M3""
---
### error:Flash Download failed - "Cortex-M3
stm32内部flash编程实验,循环擦除编程几次出现<font size="10">"error:Flash Download failed - "Cortex-M3"</font><br>
程序就下载不了了 用J-FLASH连接擦除芯片（erase chip）也不行,但用J-FLASH产生最大的测试数据就可以下载一次，之后就恢复正常了。难道是循环擦除编程使flash产生了错误，而用J-FLASH产生最大的测试数据就可以下载一次使flash的错误恢复？？？