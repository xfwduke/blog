---
layout: post
title: "AT&T 中的截取"
date: 2015-06-02 16:09:25 +0800
comments: false
categories:
  - 汇编
---

对应于 _Intel_ 语法的 {% post_link ptr-operator-yu-jie-qu PTR Operator 与截取 %}

在 _AT&T_ 中虽然没有 _PTR OPS_, 但是存在指令的字长后缀, 参考 {% post_link guan-yu-mov 关于 mov %}

利用指令后缀, 可以实现 _PTR OPS_ 中 _BYTE PTR , WORD PTR , DWORD PTR , QWORD PTR_ 一样的功能

* 和 _Intel_ 语法的 _PTR OPS_ 一样, 当汇编器可以推断出源和目的操作数字长时, 可以不用给 _mov_ 添加后缀
    * 当操作数有一个是寄存器时, 几乎都可以不需要后缀
    * 将立即数载入内存, 必须要有后缀
    * 如果使用了后缀, 就必须要与字长相符
    * 既然如此, 最好无论什么时候都用后缀, 省得糊涂
* 和 _Intel_ 语法的 _PTR OPS_ 不同的是, _AT&T_ 的字长后缀不仅可以作用于内存, 也可以作用于寄存器

<!--more-->

给出一段简单的代码, 来看看 _AT&T_ 中如何做截取

{% codeblock %}
.section .data
anum:
    .int 0xA1B2C3D4

.section .bss

.section .text

.globl _start

_start:
    movb anum + 1, %al
    movsx anum + 2, %rbx

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}
	
{% codeblock %}
(gdb) p /x $rax
$1 = 0xc3
(gdb) p /x $rbx
$2 = 0xffffffffffffffb2
{% endcodeblock %}


需要注意的是, 虽然字长后缀可以作用于寄存器, 但是对于寄存器, 是没有办法用类似上面的办法随意的截取任意部分的 —— 如从 _rax_ 中截取出第 _3_ 字节