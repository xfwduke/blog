---
layout: post
title: "数据段的命令和数据类型"
date: 2014-12-07 15:14:54 +0800
comments: false
categories: 
  - 汇编
tags: 
  - Professional Assembly Language
---

数据段定义用来存储项目的特定内存位置, 这些项目可以被程序中的指令码引用

使用 `.data` 声明的数据段的元素可以被指令码读取和写入

使用 `.rodata` 声明的数据段的元素是只读的

<!--more-->

定义数据元素的方式是

{% codeblock %}
.section .data
label:
	.cmd	data
{% endcodeblock %}

* _label_ 类似于 _C_ 的变量名称
* _.cmd_ 用于指定元素的数据类型
* _data_ 为实际的数据


{% codeblock 数据定义示例 %}
.section .data
output:
	.ascii	"Hello World\n"
numbers:
	.long	100, 150, 200
{% endcodeblock %}

### 命令和数据类型的对应关系

|命令|数据类型|
|:---:|:---:|
_.byte_ | 字节值
_.ascii_ | 文本字符串
_.asciz_ | 以空字符结尾的文本字符串
_.float_ | 单精度浮点数
_.single_ | 单精度浮点数(和 _.float_ 相同)
_.double_ | 双精度浮点数
_.short_ | _16_ 位整数
_.int_ | _32_ 位整数
_.long_ | _32_ 位整数(和 _.int_ 相同)
_.quad_ | _8_ 字节整数
_.octa_ | _16_ 字节整数

### 静态符号 (常量)

{% codeblock %}
.section .data
.equ SOFT_INT, 0x80
{% endcodeblock %}

这样的形式定义一个类似于常量的 _SOFT\_INT_ , 值是 _0x80_. 在程序中 `int $SOFT_INT` 来使用 