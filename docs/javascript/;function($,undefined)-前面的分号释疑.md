title: ;function($,undefined) 前面的分号释疑
date: 2014-05-04 14:43:05
categories: Javascript
tags: [jquery]
---

;(function($){$.extend($.fn...

一般在一些 JQuery 函数前面有分号,在前面加分号可以有多种用途:

1、防止多文件集成成一个文件后，高压缩出现语法错误。

2、这是一个匿名函数，一般js库都采用这种自执行的匿名函数来保护内部变量 (function(){})()。

3、因为undefined是window的属性，声明为局部变量之后，在函数中如果再有变量与undefined作比较的话，程序就可以不用搜索undefined到window，可以提高程序性能。

