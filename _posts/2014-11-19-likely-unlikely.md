---
layout: posts
title: "Linux中的likely()和unlikely()"
---

### [(转)Linux中的likely()和unlikely()](http://blog.chinaunix.net/uid-25409479-id-158584.html)
likely()与unlikely()在2.6内核中，随处可见，那为什么要用它们？它们之间有什么区别呢？<br>
首先明确：
>if (likely(value))等价于if (value)<br>
>if (unlikely(value))等价于if (value)

也就是说likely()和unlikely()从阅读和理解的角度是一样的。<br>
这两个宏在内核中定义如下：
<pre class="prettyprint linenums">
<linux/compiler>
require_once 'Zend/Uri/Exception.php';
require_once 'Zend/Uri/Http.php';
require_once 'Zend/Uri/Mailto.php';

abstract class Zend_Uri
{

  /**
   * Return a string representation of this URI.
   *
   * @see     getUri()
   * @return  string
   */
  public function __toString()
  {
      return $this-&gt;getUri();
  }

  static public function factory($uri = 'http')
  {
      $uri = explode(':', $uri, 2);
      $scheme = strtolower($uri[0]);
      $description = 'long
description';
      $schemeSpecific = isset($uri[1]) ? $uri[1] : '';

      // Security check: $scheme is used to load a class file,
      // so only alphanumerics are allowed.
      if (!ctype_alnum($scheme)) {
          throw new Zend_Uri_Exception('Illegal scheme');
      }
  }
}
</pre>
这里的\_\_built\_expect()函数是gcc(version >= 2.96)的内建函数,提供给程序员使用的，目的是将"分支转移"的信息提供给编译器，这样编译器对代码进行优化，以减少指令跳转带来的性能下降。<br>
\_\_buildin\_expect((x), 1)表示x的值为真的可能性更大.<br>
\_\_buildin\expect((x), 0)表示x的值为假的可能性更大.<br>
也就是说，使用likely(),执行if后面的语句的机会更大，使用unlikely(),执行else后面的语句机会更大一些。通过这种方式，编译器在编译过程中，会将可能性更大的代码紧跟着后面的代码，从而减少指令跳转带来的性能上的下降。比如：
<xmp class="prettyprint linenums">
if(likely(a>b)){
	fun1();
}
if(unlikely(a<b)){
	fun2();
}
</xmp>
<xmp class="my_xmp_class">
	这里就是程序员可以确定 a>b 在程序执行流程中出现的可能相比较大，因此运用了likely()告诉编译器将fun1()函数的二进制代码紧跟在前面程序的后面，这样就cache在预取数据时就可以将fun1()函数的二进制代码拿到cache中。这样，也就添加了cache的命中率。
	同样的，unlikely()的作用就是告诉编译器，a<b的可能性很小所以这里在编译时,将fun2()的二进制代码尽量不要和前边的编译在一块。
	咱们不用对likely和unlikely感到迷惑，须要知晓的就是 if(likely(a>b)) 和 if(a>b)在功能上是等价的，同样 if(unlikely(a<b)) 和 if(a<b) 的功能也是一样的。不一样的只是他们声称的二进制代码有所不一样，这一点咱们也可以从他们的汇编代码中看到。比如下面的代码：
</xmp>
<xmp class="prettyprint linenums">
#include <stdio.h>
#define unlikely(x) __builtin_expect(!!(x), 0)
#define likely(x) __builtin_expect(!!(x), 1)
int main()
{
	int a=2,b=4;
	if( unlikely(a<b) ){
		printf("in the unlikely,is not your expecting!\n");
	} 
	else{
		printf("in the unlikely, is your expecting\n");
	}
	if( likely(a<b) ){
		printf("in the likely, is your expecting\n");
	}
	return 0;
}
</xmp>
执行结果:<br>
in the unlikely,is not your expecting!<br>
in the likely, is your expecting<br>
总之，likely和unlikely的功能就是添加cache的命中率，提高系统执行速度。<br>