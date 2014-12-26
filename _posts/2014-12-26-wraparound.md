---
layout: posts
title: "Wraparound现象"
---
# {{ page.title }}
<xmp class="prettyprint linenums">INT16U u16a = 40000; /* unsigned short / unsigned int */
INT16U u16b = 30000; /* unsignedshort / unsigned int */
INT32U u32x; /* unsigned int / unsigned long  */
u32x = u16a + u16b; /*u32x = 70000 or 4464 ?      */</xmp>
<xmp class="my_xmp_class">期望的结果是70000，但是赋给 u 的值在实际中依赖于 int 实现的大小。如果 int 实现的大小是32 位，那么加法就会在有符号的 32 位数值上运算并且保存下正确的值。如果 int 实现的大小仅是16 位，那么加法会在无符号的 16 位数值上进行，于是会发生折叠（wraparound ）现象并产生值4464（70000%65536 ）。无符号数值的折叠（wraparound）是经过良好定义的甚至是有意的；但也会存在潜藏的混淆。


linux+v2.6.24/arch/x86/boot/header.S中第256行xorw    %dx, %dx        # Prevent wraparound，与8086/8088的wraparound现象，还有上面的例子都提到wraparound意思都是一样的，都有折叠的意思。</xmp>