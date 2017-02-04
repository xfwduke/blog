---
layout: post
title: "十进制转换成二进制"
date: 2015-02-13 22:29:21 +0800
comments: false
categories:
  - 汇编
tags: 
  - Professional Assembly Language
---

在研究 _CF OF_ 位时写一些小程序来验证什么时候这两个标志会被 _turned on_ , 程序的输出希望把数字按照二进制显示, 但是汇编和 _C_ 库没有原生的方法, 需要自己想办法

研究了下, 可以使用左移操作结合 _CF_ 实现

## 左移操作

{% codeblock %}
sal destination
sal %cl, destination
sal $num, destination
{% endcodeblock %}

* _destination_ 只能是寄存器
* 只有 _destination_ 操作数时, 左移一位
* 有两个操作数时, 将 _destination_ 左移第一个操作指定的位数
* _CF_ 位会保留最后一次左移时被丢弃的位的值 —— 即左移前的最高位的值

<!--more-->

## 实现



{% codeblock %}
.section .data

result_s: .asciz "%d%d%d%d%d%d%d%d\n"

.section .text

.globl _start

_start:
    movl $0, %ebx
    movl $172, %eax

    _loop:
    salb %al

    jc _get_one
    pushl $0
    jmp _output
    _get_one:
    pushl $1

    _output:
    inc %ebx
    cmpl $8, %ebx
    je _exit
    jmp _loop


_exit:
    pushl $result_s
    call printf
    movl $1, %eax
    movl $0, %ebx
    int $0x80
{% endcodeblock %}

上面这段代码是一个简单的实现

* _%eax_ 是需要转换成二进制格式的数
* 程序实现为只转换 _%al_ , 即 _8 bit_
* _%ebx_ 是循环计数器
* 执行 _8_ 次左移, 每次都把 _CF_ 即 _%al_ 最高位的值入栈
* 最后调用 _printf_ 输出即可