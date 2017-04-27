---
title: 'C 语言中 %d,%o,%f,%e,%x 的意义'
date: 2017-04-23 16:17:36
categories: 编程
tags: 
- C语言
---

<h1 style="text-align:center">C-语言中-d-o-f-e-x-的意义</h1>

<p><span style="font-size:14px">格式说明由 &ldquo;％&rdquo; 和格式字符组成，如％d％f 等。它的作用是将输出的数据转换为指定的格式输出。格式说明总是由 &ldquo;％&rdquo; 字符开始的。不同类型的数据用不同的格式字符。&nbsp;<br />
格式字符有 d,o,x,u,c,s,f,e,g 等。&nbsp;</span><br />
如</p>

<div>
<pre style="margin-left:40px">
％d 整型输出，％ld 长整型输出，

％o 以八进制数形式输出整数，

％x 以十六进制数形式输出整数，

％u 以十进制数输出 unsigned 型数据 (无符号数)。

％c 用来输出一个字符，

％s 用来输出一个字符串，

％f 用来输出实数，以小数形式输出，

％e 以指数形式输出实数，

％g 根据大小自动选 f 格式或 e 格式，且不输出无意义的零。

scanf(控制字符，地址列表) 
格式字符的含义同 printf 函数，地址列表是由若干个地址组成的表列，可以是变量的地址，或字符串的首地址。如 scanf(&quot;％d％c％s&quot;,&amp;a,&amp;b,str)；</pre>
</div>
