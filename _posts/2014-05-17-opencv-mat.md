---
layout: posts
title: "编程最基本的问题之数据访问"
---

# 编程最基本的问题之数据访问
## <font color="blue">背景</font>
最开始学习opencv的时候没太注意对基本的图像容器Mat，直到后来每次涉及到最基本的问题时总要没有思路的回过头来看，比如最简单的定义一个图像、访问图像的某个像素，所以这里稍微总结一下。
## <font color="blue">Mat定义</font>

1. Mat A=imread(argv[1], CV_LOAD_IMAGE_COLOR);
2. Mat D (A, Rect(10, 10, 100, 100) ); // using a rectangle
3. Mat F = A.clone();
4. A.copyTo(G);
5. Mat M(2,2, CV_8UC3);
6. Mat M(2,2, CV_8UC3, Scalar(0,0,255));
7. int sz[3] = {2,3,4};Mat L(3,sz, CV_8UC(3), Scalar::all(0));
8. Mat C = (Mat_<double>(3,3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);
9. IplImage\* img = cvLoadImage("greatwave.png", 1);Mat mtx(img); // convert IplImage\* -> Mat

解释：

1. 把mat看做磁盘上的文件，直接读取
2. 把mat看做已知mat（图像）的一个矩形部分，直接通过Rect定义即可
3. 把mat看做已知mat的clone，clone方法<font color="red">返回mat</font>
4. 把mat看做已知mat的copy，copyTo方法<font color="red">需要传入mat</font>
5. <font color="red">把mat看做一个二维数组，需要注意数组每个元素的类型定义：<br>
CV_[The number of bits per item][Signed or Unsigned][Type Prefix]C[The channel number]</font>
6. 同上，只是对数组中每个元素初始化值
7. 比较不常用的方式，第一个实参3代表维度，第二个实参代表每个维度尺寸的数组，第三个实参Scalar::all(0)代表给每一个元素初始化，其实就是定义了一个3维（2\*3\*4）的mat并且每个元素是CV_8UC(3)类型
8. 小矩阵逗号分隔初始化函数
9. <font color="red">为已存在IplImage指针创建信息头,或者说IplImage\*转换为Mat，也算是很正规的构造函数了</font>

## <font color="blue">Mat像素的访问</font>
图像矩阵的大小取决于我们所用的颜色模型，确切地说，取决于所用通道数。如果是灰度图像，矩阵就会像这样：
![img](/images/opencv/灰度mat.png)<br>
而对多通道图像来说，矩阵中的列会包含多个子列，其子列个数与通道数相等。例如，RGB颜色模型的矩阵：
![img](/images/opencv/彩色mat.png)<br>
注意到，子列的通道顺序是反过来的：BGR而不是RGB。很多情况下，因为内存足够大，可实现连续存储，因此，图像中的各行就能一行一行地连接起来，形成一个长行。连续存储有助于提升图像扫描速度，我们可以使用 isContinuous() 来去判断矩阵是否是连续存储的. 相关示例会在接下来的内容中提供。

<font color="blue">第一种方法：</font>
<xmp class="prettyprint linenums">
Mat& ScanImageAndReduceC(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() != sizeof(uchar));     

    int channels = I.channels();

    int nRows = I.rows * channels; 
    int nCols = I.cols;

    if (I.isContinuous())
    {
        nCols *= nRows;
        nRows = 1;         
    }

    int i,j;
    uchar* p; 
    for( i = 0; i < nRows; ++i)
    {
        p = I.ptr<uchar>(i);
        for ( j = 0; j < nCols; ++j)
        {
            p[j] = table[p[j]];             
        }
    }
    return I; 
}
</xmp>
注意：

1. int nRows = I.rows * channels;<br>
两点：<br>
1.I.rows整体上把图像看做是一个二维的，其基本元素是像素，而I.rows * channels也是二维的角度，只不过其基本元素是每个像素的基本RGB分量。<br><br>
2.可以看看上图似乎nCols = I.cols * channels;更合适，这里就需要注意一下了,一方面opencv这样存储，另一方面用数据表示数据的时候，尽量把数据拆分成行更常用，而不是拆分成列，比如把二维数组看成是一行一行的（由一维数组组成）。
2. if (I.isContinuous())<br>
如果isContinuous()为真，其实可以把图像看做是一维数组，所以另nRows = 1;而nCols *= nRows;
3. <xmp style="white-space: pre-wrap; word-wrap: break-word;">关于I.ptr<uchar>(i)</xmp>
把图像看做是二维数组，切最基本的元素是RGB分量。所以<xmp style="white-space: pre-wrap; word-wrap: break-word;">I.ptr<uchar>(i)</xmp>返回的是第i行的指针（注意并不是图像上第i行的指针）。是否有<xmp style="white-space: pre-wrap; word-wrap: break-word;">I.ptr<Vec3b>(i)</xmp>呢？？

<font color="blue">第二种方法：</font>
<xmp class="prettyprint linenums">
Mat& ScanImageAndReduceIterator(Mat& I, const uchar* const table)
{
    // accept only char type matrices
    CV_Assert(I.depth() != sizeof(uchar));     
    
    const int channels = I.channels();
    switch(channels)
    {
    case 1: 
        {
            MatIterator_<uchar> it, end; 
            for( it = I.begin<uchar>(), end = I.end<uchar>(); it != end; ++it)
                *it = table[*it];
            break;
        }
    case 3: 
        {
            MatIterator_<Vec3b> it, end; 
            for( it = I.begin<Vec3b>(), end = I.end<Vec3b>(); it != end; ++it)
            {
                (*it)[0] = table[(*it)[0]];
                (*it)[1] = table[(*it)[1]];
                (*it)[2] = table[(*it)[2]];
            }
        }
    }
    
    return I; 
}
</xmp>
说到Iterator就要注意基本单位，所以如果I.channels()为1，最基本的单位就是char，所以用![img](/images/opencv/MatIterator_uchar.jpg)类型，如果I.channels()为3，最基本的单位是三分量的像素，所以用![img](/images/opencv/MatIterator_Vec3b.jpg)类型。