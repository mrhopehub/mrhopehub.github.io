---
layout: posts
title: "eclipse 常用"
---
# {{ page.title }}
<xmp class="my_xmp_class">
</xmp>
<xmp class="prettyprint linenums">
</xmp>

1.缩进<blockquote></blockquote>或者> >> >>>
2.标题# 文章题目 ## 一级标题1 ### 二级标题1.1 #### 三级标题1.1.1，从二级标题开始缩进
3.格式化文本<xmp class="my_xmp_class"></xmp>(只有黑体，xmp内部的标签不起作用)
4.自由文本<font></font>或者<xmp style="color: red; font-size: 14px;" class="my_xmp_class"><!--可改变大小、颜色的格式化文本--></xmp>
5.代码<xmp class="prettyprint linenums"></xmp>
6.滚动代码，21行才能用<div class="scroll_code"><xmp class="prettyprint linenums"></xmp></div>
7.![向导](/images/markdown语法/mdcheatsheet.png)<br><!--图片，绝对地址-->
8.图片、图片说明居中<div align=center><img src="/images/" alt=""></div>
9.站内文章链接[Name of Link]({% post_url 2010-07-21-name-of-post %})
10.jekyll官网http://jekyllrb.com/docs/home/


HTML默认样式http://unmi.cc/html-tags-default-style-in-browser/
 
html, address,blockquote,body, dd, div,dl, dt, fieldset, form,frame, frameset,h1, h2, h3, h4,h5, h6, noframes,ol, p, ul, center,dir, hr, menu, pre { display: block }
li { display: list-item }
head { display: none }
 
table { display: table }
tr { display: table-row }
thead { display: table-header-group }
tbody { display: table-row-group }
tfoot { display: table-footer-group }
col { display: table-column }
colgroup { display: table-column-group }
td, th { display: table-cell; }
caption { display: table-caption }
th { font-weight: bolder; text-align: center }
caption { text-align: center }
 
body { margin: 8px; line-height: 1.12 }
 
h1 { font-size: 2em; margin: .67em 0 }
h2 { font-size: 1.5em; margin: .75em 0 }
h3 { font-size: 1.17em; margin: .83em 0 }
h4, p, blockquote, ul, fieldset, form, ol, dl, dir, menu { margin: 1.12em 0 }
h5 { font-size: .83em; margin: 1.5em 0 }
h6 { font-size: .75em; margin: 1.67em 0 }
h1, h2, h3, h4, h5, h6, b,strong { font-weight: bolder }
 
blockquote { margin-left: 40px; margin-right: 40px }
i, cite, em,var, address { font-style: italic }
 
pre, tt, code, kbd, samp { font-family: monospace }
pre { white-space: pre }
 
button, textarea, input, object, select { display:inline-block; }
big { font-size: 1.17em }
small, sub, sup { font-size: .83em }
sub { vertical-align: sub }
sup { vertical-align: super }
table { border-spacing: 2px; }
thead, tbody, tfoot { vertical-align: middle }
td, th { vertical-align: inherit }
s, strike, del { text-decoration: line-through }
hr { border: 1px inset }
ol, ul, dir, menu, dd { margin-left: 40px }
ol { list-style-type: decimal }
ol ul, ul ol, ul ul, ol ol { margin-top: 0; margin-bottom: 0 }
u, ins { text-decoration: underline }
 
br:before { content: "\A" }
:before, :after { white-space: pre-line }
center { text-align: center }
abbr, acronym { font-variant: small-caps; letter-spacing: 0.1em }
:link, :visited { text-decoration: underline }
:focus { outline: thin dotted invert }
BDO[DIR="ltr"] { direction: ltr; unicode-bidi: bidi-override }
BDO[DIR="rtl"] { direction: rtl; unicode-bidi: bidi-override }
*[DIR="ltr"] { direction: ltr; unicode-bidi: embed }
*[DIR="rtl"] { direction: rtl; unicode-bidi: embed }
 
@media print {
    h1 { page-break-before: always }
    h1, h2, h3,    h4, h5, h6 { page-break-after: avoid }
    ul, ol, dl { page-break-before: avoid }
}