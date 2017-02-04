---
layout: post
title: "打印 ASCII 码表"
date: 2014-12-05 21:06:21 +0800
comments: true
categories: 
  - 汇编
tags: 
  - Programming from the Ground Up Book
---

看了书上的例子, 尝试着写了个打印 _ASCII_ 码表的程序
<!--more-->



{% codeblock 打印 ASICC 码表 %}
.section .data
.section .text
.globl _start

_start:
    movl $0, %ecx
    pushl %ecx

_loop:
    movl $4, %eax
    movl $1, %ebx
    movl $1, %edx
    movl %esp, %ecx
    int $0x80

    popl %ecx
    incl %ecx
    cmpl $256, %ecx
    je _exit
    pushl %ecx
    jmp _loop

_exit:
    pushl $10
    movl %esp, %ecx
    movl $4, %eax
    movl $1, %ebx
    movl $1, %edx
    int $0x80
    movl $1, %eax
    int $0x80
{% endcodeblock %}

## _sys\_write_

> 系统调用号 4


| Name | \%eax | \%ebx | \%ecx | \%edx |
|:----:|:----:|:----:|:----:|:----:|
| sys_write | 4 | unsinged int | const char* | size_t |

这个系统调用, 作用就是写文件, 所以也可以向屏幕打印内容. 重点就是第二个参数必须是一个指针

这样的用法, 在 _c_ 里面是非常简单的, 但是在汇编里面就有些麻烦了, 上面代码的实现方法是

* 把要输出的内容用 _pushl_ 压到栈中, _%esp_ 永远指向栈顶
* 取得 _%esp_ 的值送到 _%ecx_, 则 _%ecx_ 就指向了要输出的内容
* 为了省资源, 每次指向完系统调后, 调用 _popl_ 扔掉已经输出的内容

