---
layout: post
title: "汇编函数"
date: 2015-03-31 20:00:38 +0800
comments: false
categories:
  - 汇编
tags:
  - Professional Assembly Language
---

主要是些概念, 基本就是直接照搬 《汇编语言程序设计》 了

汇编语言程序中创建函数需要 _3_ 个步骤

* 定义需要的输入值 —— 参数
* 定义对输入值执行的操作 —— 函数体
* 定义如何生成输出值以及如何把输出值传递给发出调用的程序 —— 返回值

<!--more-->

## 定义输入值

有 _3_ 种办法可以传递参数

* 使用寄存器
* 使用全局变量
* 使用栈

## 定义函数处理

为了在 _GNU_ 汇编器中定义函数, 必须在程序中把函数名称声明为标签, 格式是

{% codeblock %}
.type my_func_name, @function

my_func_name:
    ...
    ...
    ret
{% endcodeblock %}

函数的结束由 _ret_ 指令定义, 执行到 _ret_ 指令时, 程序控制返回主程序, 返回的位置是 __紧跟__ 在调用函数的 _call_ 指令后面的指令

## 定义输出值
有多种方式完成传送结果的工作, 最常见的是

* 把结果放在一个或者多个寄存器中
* 把结果放在全局变量内存位置中

## 完整的函数

以计算圆面积为例子来写一个完整的函数

{% codeblock %}
.type area, @function

area:
    fldpi
    imull %ebx, %ebx
    movl %ebx, value
    filds value
    fmulp %st(0), %st(1)
    ret
{% endcodeblock %}
	
* 函数的参数 —— 圆半径放在 _ebx_ 中
* 函数的结果 —— 圆面积放在全局变量 _value_ 中