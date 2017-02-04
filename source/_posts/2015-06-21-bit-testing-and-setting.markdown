---
layout: post
title: "Bit testing and setting"
date: 2015-06-21 22:17:22 +0800
comments: false
categories:
  - 汇编
---

之前学习的指令操作的对象最小都是 _1 byte_

这一组位操作指令, 操作对象最小的 _1 bit_

这一部分主要包含

* _Bit Test and Modify Instructions_
* _Bit Scan Instructions_
* _Byte Set On Condition Instructions_

<!--more-->

## _Bit Test and Modify Instructions_

书中[^1]提到 _3_ 个指令, 在查阅 _Intel_ 官方文档[^2][^3]时发现实际上有 _4_ 个

_Instruction_|_Effect on CF Flag_|_Effect on Selected Bit_
:----:|:----:|:----:
_BT ( Bit Test )_|_CF flag_ $\leftarrow$ _Selected Bit_|_No effect_
_BTS ( Bit Test and Set )_|_CF flag_ $\leftarrow$ _Selected Bit_|_Selected Bit_ $\leftarrow$ _1_
_BTR ( Bit Test and Reset )_|_CF flag_ $\leftarrow$ _Selected Bit_|_Selected Bit_ $\leftarrow$ _0_
_BTC ( Bit Test and Complement )_|_CF flag_ $\leftarrow$ _Selected Bit_|_Selected Bit_ $\leftarrow\lnot$_Selected Bit_

这一组指令都有两个操作数, 分别叫做 _BitBase_ 和 _BitOffset_

* _BitBase_ 可以是 _16 ~ 64 bit_ 寄存器
* _BitOffset_ 可以是 _16 ~ 64 bit_ 寄存器以及 _8 bit_ 立即数

在 _Intel_ 语法中的基本形式是

{% codeblock %}
BT BitBase, BitOffset
{% endcodeblock %}
	
_AT&T_ 中要调换个位置

举个例子,  `BT [anum], 7` 的意思是测试以内存地址 _&anum_ 为起始偏移为 _7_ ( 从 _0_ 开始) 的位

## _Bit Scan Instructions_

* _BSF : bit scan forward_
* _BSR : bit scan reverse_

有两个操作数, 在 _source_ 中找 _1_, 把找到的第一个 _1_ 的位置 ( 从 _0_ 开始)放到 _destination_ 中. 更详细的看文档[^2][^3]

## _Byte Set On Condition Instructions_

_Setcc_ 指令在书中[^1]被归类到位操作, 但是在文档[^2]中的标题名是 _Byte Set_

指令的操作数确实是 _Byte_ , 但都是在某种条件下将操作数的值置为 _1_ 或 _0_ —— 只 _Set 1 bit_

在文档[^3]中 _Setcc_ 指令有一大版, 参考 {% post_link eflags-condition-codes EFLAGS Condition Codes %}  , 这一大版实际就是带有各种 _Conditional Code_ 的指令

_Setcc_ 指令只有一个操作数, 并且严格限制为 _8 bit_ 的寄存器或者内存 —— 即只能是 _1 byte_

当符合 _cc_ 代表的条件时操作数的值被置为 _1_ , 否则置为 _0_

{% codeblock %}
.section .data
anum:
.byte 0b10001111

.section .bss

.section .text

.globl _start

_start:
    xor %al, %al
    shlb $1, anum
    setc %al
    xor %al, %al
    movb $11, %al
    shlb $1, anum
    setc %al

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}


* 执行 _13_ 行左位移后 $CF=1$, 所以在 _14_ 行将 $al\leftarrow1$
* 执行 _17_ 行左位移后 $CF=0$, 所以在 _18_ 行将 $al\leftarrow0$ ( _16_ 行的赋值是为了方便观察 _setc_ 的行为)

[^1]:《Introduction to 64 Bit Assembly Programming for Linux and OS X》 Ray Seyfarth
[^2]:《Intel 64 and IA-32 Architectures Software Developer’s Manual Vomume 1》
[^3]:《Intel 64 and IA-32 Architectures Software Developer’s Manual Vomume 2》