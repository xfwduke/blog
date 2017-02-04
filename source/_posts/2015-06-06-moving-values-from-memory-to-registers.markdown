---
layout: post
title: "Moving values from memory to registers"
date: 2015-06-06 15:12:54 +0800
comments: false
categories:
  - 汇编
---

无论内存地址是直接的明文地址, 变量名还是放在寄存器中, 差别并不大. 

这类操作的特点是, 源操作数字长无法直接确认, 目的操作数字长是可以确认的.

* 内存地址只是用来确认一块连续内存块的起始, 内存块的长度是可变的
* 寄存器虽然有很多个, 但其字长总是固定的

<!--more-->

## 源字长大于目的字长

由于在 _Intel_ 语法中有 _PTR OPS_ , 这种情况 _Intel_ 语法比 _AT&T_ 要复杂

### _AT&T_

多出来的高位数据被丢弃了

### _Intel_

在不使用 _PTR OPS_ 时, 和 _AT&T_ 一样把多出来的高位丢弃

使用 _PTR OPS_ 可以从源截取出特定字长(一般和寄存器字长相等)的数据

## 源字长小于目的字长

这种情况更复杂一些

* 可以只使用 _mov_ 
* 可以使用位扩展指令代替 _mov_

### 只使用 _mov_

汇编并没有真正抽象出 `变量` 这个概念, 在 `data 段` 定义的所谓变量, 只是用来指向特定长度的内存块, 甚至是这个长度都没有强制性

只使用 _mov_ 将数据载入寄存器, 总会以指令中给出的内存地址位起始将足够字长的数据载入寄存器, 这一点在 _AT&T_ 和 _Intel_ 中是一样的

{% codeblock %}
segment .data
anum dw 0x1122
bnum dw 0x3344

segment .bss

segment .text

global _start

_start:
    mov eax, [anum]

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock %}
(gdb) p /x $rax
$1 = 0x33441122
{% endcodeblock %}
	
_anum bnum_  均定义为 _16 bit_ 的变量 , 程序中试图将 _anum_ 的值载入 _eax_

由于 _eax_ 是 _32 bit_ 寄存器, 所以结果是还将 _bnum_ 载入了 _eax_ 

### 使用位扩展指令

_movsx movzx_ 没什么好说的, 非常符合预期的结果

唯一要提到的是, 在 _Intel_ 语法中源操作数字长时 _dword_ 时, 要使用 _movsxd_ 和 _movzxd_

### 位扩展与 _PTR OPS_

这是 _Intel_ 语法特有的

{% codeblock %}
segment .data
anum dw 0x11B2

segment .bss

segment .text

global _start

_start:
    movsx rax, byte [anum]
    movzx rbx, byte [anum]
    movsx rcx, byte [anum + 1]
    movzx rdx, byte [anum + 1]

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock %}
(gdb) p /x $rax
$1 = 0xffffffffffffffb2
(gdb) p /x $rbx
$2 = 0xb2
(gdb) p /x $rcx
$3 = 0x11
(gdb) p /x $rdx
$4 = 0x11
{% endcodeblock %}
	
使用位扩展指令和 _PTR OPS_ 可以随意截取出需要的部分做位扩展, 扩展的符号位是参考截取出来的部分的符号位  