---
layout: posts
title: "exit重定义"
---
# {{ page.title }}
## vs2010提示错误
<xmp class="prettyprint linenums">1>c:\program files\microsoft visual studio 10.0\vc\include\stdlib.h(353): error C2381: “exit”: 重定义；__declspec(noreturn) 不同
1>          d:\vs\opengl\opengl\opengl_sdk\include\glut.h(146) : 参见“exit”的声明</xmp>
## 解决方法
<xmp class="my_xmp_class">引用0penGL头文件放在最后。
如下代码所示：</xmp>
<xmp class="prettyprint linenums">#include <stdlib.h>
#include <GL/glut.h></xmp>