---
layout: post
title: "PTR Operator 与截取"
date: 2015-06-02 15:15:21 +0800
comments: false
categories:
  - 汇编
---

_PTR Operator_ 在 _Intel_ 语法中算是一个比较重要的概念[^1]

{% codeblock %}
Syntax:
	type PTR name
	
type filed:
	1. BYTE
	2. WORD
	3. DWORD
	4. QWORD
	5. TBYTE
	6. NEAR
	7. FAR

name filed:
	1. A variable name
	2. A label name
	3. An address or register expression
	4. An integer that represents an offset
{% endcodeblock %}

需要重点注意的是 : ___PTR OPS_ 只能作用于内存对象__

_PTR OPS_ 的作用是 : 将 `name field` 所指内存位置的数据看做是 `type filed` 类型
	
由于目前只学到了 _mov_ , _PTR OPS_ 也只学了前 _4_ 个 `type filed` , 所以先只讨论这些

<!--more-->

## 作用于 _Constant_

其实 _PTR OPS_ 是可以作用于常量的, 只是汇编器会有告警, 结果页符合预期 —— 不得不说的是, 常量真的没什么用

## 直接寻址

在指令中直接出现内存地址 , 这其实包括了在指令中使用数字形式的内存地址和使用变量名

但是在这里特指直接使用数字形式的地址

{% codeblock %}
mov rax, word [0x11]
{% endcodeblock %}

这样的用法虽然合法, 但是很少有在编码阶段就知道明确地址的数据, 所以代码的执行结果大多会是

`Segmentation Fault`

## 作用于变量

特指在 _data_ 段定义的变量

{% codeblock %}
segment .data
anum dd 0x11223344

segment .bss

segment .text

global _start

_start:
    mov ax, word [anum]
    mov bx, word [anum + 1]

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock %}
(gdb) p /x $ax
$2 = 0x3344
(gdb) p /x $bx
$3 = 0x2233
{% endcodeblock %}


## 和寄存器一起用


_PTR OPS_ 不能直接作用于寄存器, 但是可以作用于寄存器值所指向的内存地址 —— 即把寄存器看做一个指针{% codeblock %}
segment .data
anum dd 0x11223344

segment .bss

segment .text

global _start

_start:
    mov rcx, anum
    mov ax, word [rcx]
    mov bx, word [rcx +1]

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock %}(gdb) p &anum
$1 = (<data variable, no debug info> *) 0x6000dc <anum>
(gdb) p /x $rcx
$2 = 0x6000dc
(gdb) p /x $ax
$3 = 0x3344
(gdb) p /x $bx
$4 = 0x2233{% endcodeblock %}


## _error: mismatch in operand sizes_


* _mov_ 的操作数必定是寄存器
* _PTR OPS_ 只能作用于内存对象

一旦在 _mov_ 中使用了 _PTR OPS_ , 则寄存器的字长必须和 _PTR OPS_ 的一致, 否则就会报错

{% codeblock %}
mov ax, dword [anum]
mov dword [anum], rax
{% endcodeblock %}

像这样的用法是不行的, 因为 _ax_ 应该对应 _word_ , 而 _rax_ 对应 _qword_




[^1]: ASM86 Language Referance Manual : 4-15
