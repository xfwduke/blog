---
layout: post
title: "Moving values from a register to memory"
date: 2015-06-07 14:47:57 +0800
comments: false
categories:
  - 汇编
---

将寄存器作为源操作数, 内存作为目的操作

与将数据载入寄存器不同的是

* 不能使用位扩展
* 当源字长大于目的时, 更加危险

<!--more-->

## 源字长小于目的字长

由于源操作数寄存器的字长是固定的, 所以只会影响内存中对应位的数据, 多出来的位不受影响 —— 在 _AT&T_ 和 _Intel_ 中是一样的

{% codeblock %}
.section .data
anum:
    .quad -1

.section .bss

.section .text

.globl _start

_start:
    xor %rax, %rax
    movb %al, anum

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}
	
{% codeblock %}
(gdb) x /8bx &anum
0x6000ca:       0x00    0xff    0xff    0xff    0xff    0xff    0xff    0xff
{% endcodeblock %}
	
## 源字长大于目的字长

由于内存是一片很大的连续空间, 而汇编又没有真正意义的 `变量` 概念, 所以寄存器中的值只会被完全载入到目的操作数指向的连续内存空间, 即使有预期之外的内存空间被覆盖, 汇编器也不会给出任何告警或者错误, 只有在程序运行时才能发现有异常

{% codeblock %}
segment .data
anum dw 0x1122
bnum dw 0x3344
cnum dw 0x5566
dnum dw 0x7788

segment .bss

segment .text

global _start

_start:
    xor rax, rax
    mov [anum], rax

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock %}
(gdb) x /8bx &anum
0x6000d4 <anum>:        0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
(gdb) x /2bx &anum
0x6000d4 <anum>:        0x00    0x00
(gdb) x /2bx &bnum
0x6000d6 <bnum>:        0x00    0x00
(gdb) x /2bx &cnum
0x6000d8 <cnum>:        0x00    0x00
(gdb) x /2bx &dnum
0x6000da <dnum>:        0x00    0x00
{% endcodeblock %}
	
这段代码原本的意图是

> 将 _rax_ 的值载入到 _anum_ 中

但是由于 _rax_ 是 _64 bit_ 寄存器, 而 _anum_ 对应 _16 bit_ 内存, 所以其后总共 _64 bit_ 都被覆盖了 —— 这样的结果是非常危险的, 但是仔细的利用倒是能方便的给一大块内存赋值