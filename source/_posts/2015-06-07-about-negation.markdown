---
layout: post
title: "About Negation"
date: 2015-06-07 20:46:15 +0800
comments: false
categories:
  - 汇编
---

`neg` 取反操作是一个非常简单的操作, 但又是一个演示 _AT&T_ 字长后缀以及 _Intel PTR OPS_ 很好的例子

* 只有一个操作数
* 将操作数的值取反并写入操作数中
* 当结果为负数时, _SF_ 置位
* 当结果为 _0_ 时, _ZF_ 置位

<!--more-->

直接对操作数取反没什么好看的

重点来看看对操作数中的一部分取反 —— 操作数是寄存器的也不看了, 主要看操作数是内存的情况 —— 因为对于寄存器自由度并不大 ,参考 {% post_link ptr-operator-yu-jie-qu PTR Operator 与截取 %} 以及 {% post_link attzhong-de-jie-qu AT&T 中的截取 %}

直接给两种语法下的代码示例就行了

{% codeblock %}
.section .data
anum:
    .int 0x11223344

.section .bss

.section .text

.globl _start

_start:
    negb anum
    negw anum + 1

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}
	
{% codeblock %}
segment .data
anum dd 0x11223344

segment .bss

segment .text

global _start

_start:
    neg byte [anum]
    neg word [anum + 1]

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}