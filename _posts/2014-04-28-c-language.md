---
layout: posts
title: "理解C中的声明"
---

# 理解C中的声明
A    <font color="red">从声明中找出","，</font>如果存在","，则说明涉及到函数，因为只有函数的声明中存在","（当然了，除了像char a,b,c;这样的声明情况）；<br>
B    假如有","的话，","两边的是函数参数。<br>
### C    <font color="red">找出非关键字,从这里开始分析。</font><br>
D    优先级从高到低
<blockquote>
    	D1	声明中被括号括起来的那部分<br>
		D2	后缀操作符	()说明是个函数，[]说明是个数组<br>
		D3	前缀，星号"*"表示指针
</blockquote>
E    如果const和（或）volatile关键字的后面紧跟类型说明符（如int,long等），那么它作用于类型说明符，	在其他情况下，const和（或）volatile关键字作用于它左边的指针星号<br>
F    最终这个到底是定义了数组、指针变量、还是函数原型声明<br><br><br>
举例：<font color="red">void(\* signal(int signr,void(\* handler)(int)))(int);</font>//linux0.11源码include/signal.h文件第55行<br>
举例：<br>
<xmp class="prettyprint linenums">
#include <stdio.h>
int sum(int sum1,int sum2);
int main()
{
    int (*(*p))(int a,int b);
	int (*fp)(int,int);
	fp=sum;
	p=&fp;
	printf("%d\n",(*(*p))(1,2));
	printf("%d\n",(*fp)(2,3));
	return 0;
}

int sum(int sum1,int sum2)
{
	return sum1+sum2;
}
</xmp>
<br><br>
# 划清指针与数组的界限
指针默认没有维数<br>
<font color="blue">数组名是嵌套与维相结合的指针</font><br>
<blockquote>
如对于一维数组就没有嵌套的意思，而对于二维数组就可以这样理解。<br>
如int a[10][10];*(a+i)表示第i行，*(*(a+i)+j)就表示第i行第j个元素<br>
注意区别a与char *p,char **p,char (*p)[10]的区别就在嵌套与维数，<br>char *p没有嵌套没有维数,char **p有嵌套没有维数，char (*p)[10]有嵌套有维数
</blockquote>
<font color="blue">而对于一维数组来说没有嵌套与维数可言，所以在一维数组名几乎等价于指针</font><br>
<font color="red">
所以当面对datatype \*\*p一般不要与数组的维数联系起来，多半应该与结构体联系起来（linux0.11源码中的sleep_on函数、wait_on_buffer函数），注意与上面int (\*(\*p))(int a,int b)的比较（虽然本质上没有什么不同）</font>