---
layout: posts
title: "createTrackbar与printf的比较(与最简单的函数比较，方便记忆)"
---

#### createTrackbar与printf的比较(与最简单的函数比较，方便记忆)
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
int cvCreateTrackbar(const char*trackbar_name, const char* window_name, int* value, int count,CvTrackbarCallbackon_change=NULL);

printf("this is int printf %d",n);

比较：
1.const char*trackbar_name, const char* window_name, 相当于printf中的""中的内容。
2.int* value, int count,CvTrackbarCallbackon_change=NULL，相当于printf中的n。

小总结
遇到新的函数原型，那常用的函数原型、性质与之比较，方便记忆，但是要注意这两个函数到底是否相似，以免混淆。
</xmp>