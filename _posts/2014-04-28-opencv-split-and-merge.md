---
layout: posts
title: "opencv之通道的分离(split)与合并(merge)"
---

# {{ page.title }})
<xmp style="white-space: pre-wrap; word-wrap: break-word;">
分离、合并通道

分离通道：
import cv2  
import numpy as np  
img = cv2.imread("D:/cat.jpg")  
b, g, r = cv2.split(img)  
cv2.imshow("Blue", r)  
cv2.imshow("Red", g)  
cv2.imshow("Green", b)  
cv2.waitKey(0)  
cv2.destroyAllWindows()  
其中split返回RGB三个通道，如果只想返回其中一个通道，可以这样：
b = cv2.split(img)[0]  
g = cv2.split(img)[1]  
r = cv2.split(img)[2]  

通道合并：

这里举得例子是opencv_tutorials中的傅里叶变换

Mat planes[] = {Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F)}; Mat complexI; merge(planes, 2, complexI); // 为延扩后的图像增添一个初始化为0的通道
planes是一个二维数组，merge（planes，2，complexI）相当于把planes合并（merge）为一个二通道的二维数组
The functions merge merge several arrays to make a single multi-channel array. That is, each element of the output array will be a concatenation of the elements of the input arrays, where elements of i-th input array are treated as mv[i].channels()-element vectors.
</xmp>