---
layout: post
title: "Moving a constant into a register"
date: 2015-06-04 22:08:31 +0800
comments: false
categories:
  - 汇编
---

_constant_ 在 _AT&T_ 语法中叫做 立即数 ( _immediately number_ )

这个操作感觉其实没什么正经作用-_-

<!--more-->

## _AT&T_

__字长后缀以寄存器字长为标准__

* 立即数和寄存器的字长相等时, 没什么好说的
* 立即数字长大于寄存器时, 高位多出的位被丢弃, 汇编器会有告警

这个操作并不会出现 `立即数字长小于寄存器的情况` , 如 `movq $0x1, %rax` 会把 _0x1_ 看做一个 _64bit_ 数

## _Intel_

_Intel_ 语法下更为简单, 所有情况都是同样的语法

{% codeblock %}
mov rax, 0x11
mov bl, 0x11223344{% endcodeblock %}

常数字长大于寄存器时, 汇编器同样会告警