---
layout: post
title: "AT&T 和 Intel 的数据类型"
date: 2015-05-13 21:58:16 +0800
comments: false
categories:
  - 汇编
---

对于 `数据类型` 这个概念, _AT&T_ 和 _Intel_ 这两种不同的语法, 差别是比较大的

<!--more-->

## _Intel_

从目前看到的内容来看, _Intel_ 语法下并没有和数据类型相关的指令, 取而代之的是只需要指明数据的字长

{% codeblock %}
zero dd 0.0
one dw 1
neg1 dq -1
neg1f dd -1.0
c_a db 'a'
c_b dd 'b'
{% endcodeblock %}

{% codeblock %}
$ nasm -f elf64 test.asm -l test.lst
{% endcodeblock %}查看 _listing file_ 的内容是

{% codeblock %}
1                                  segment .data
2 00000000 00000000                zero dd 0.0
3 00000004 0100                    one dw 1
4 00000006 FFFFFFFFFFFFFFFF        neg1 dq -1
5 0000000E 000080BF                neg1f dd -1.0
6 00000012 61                      c_a db 'a'
7 00000013 62000000                c_b dd 'b'
8                                  segment .bss
9
10                                  segment .text
11
12                                  global _start
13
14                                  _start:
15
16                                  _exit:
17 00000000 B83C000000                  mov rax, 60
18 00000005 BF00000000                  mov rdi, 0
19 0000000A 0F05                        syscall
20
{% endcodeblock %}

_db dd_ 等指令只影响到变量的字长; 如何理解这些变量则受值的具体形式影响, 如 _1_ 和 _-1_ 为整型而 _-1.0_ 则为浮点型


## _AT&T_

在 {% post_link shu-ju-duan-de-ming-ling-he-shu-ju-lei-xing 数据段的命令和数据类型 %} 中有提到 _AT&T_ 语法下和数据类型相关的伪指令[^1]

[^1]:[Assembler Directives](http://tigcc.ticalc.org/doc/gnuasm.html#SEC67)

单独对比下整型和浮点型, 同样用 _as_ 打印出 _listing file_ [^2]

[^2]:[Enabling Listings](http://tigcc.ticalc.org/doc/gnuasm.html#SEC13)

{% codeblock lang:bash %}
$ as -a test.s
{% endcodeblock %}

{% codeblock %}
GAS LISTING test.s 			page 1


   1              	.section .data
   2 0000 000080BF 	neg1f: .single -1
   3 0004 FFFFFFFF 	neg1:	.int -1
   4 0008 0100     	ones:	.short 1
   5
   6              	.section .bss
   7
   8              	.section .text
   9
  10              	.globl _start
  11
  12              	_start:
  13
  14              	_exit:
  15 0000 48C7C03C 	    movq $60, %rax
  15      000000
  16 0007 48C7C700 	    movq $0, %rdi
  16      000000
  17 000e 0F05     	    syscall
  18

GAS LISTING test.s 			page 2


DEFINED SYMBOLS
              test.s:2      .data:0000000000000000 neg1f
              test.s:3      .data:0000000000000004 neg1
              test.s:4      .data:0000000000000008 ones
              test.s:12     .text:0000000000000000 _start
              test.s:14     .text:0000000000000000 _exit

NO UNDEFINED SYMBOLS
{% endcodeblock %}


伪指令 _.single .short .int_ 等不仅仅影响到变量的字长, 同时还决定了如何理解变量的值
	
可以试试定义 

{% codeblock %}
one: .int 1.0
{% endcodeblock %}

是会报错的

## 对比

* _Intel_ 语法下的数据定义指令只指定数据的字长

操作符|含义|字长
:----:|:----:|:----:
_db_ | _data byte_ | _8 bits_
_dw_ | _data word_ | _16 bits_
_dd_ | _data double worlds_ | _32 bits_
_dq_ | _data quad worlds_ | _64 bits_

* 而 _AT&T_ 语法则用不同的伪指令来定义对应的数据类型

两种不同的语法并没有绝对的优劣, 但还是有各自的特点

* _Intel_ 语法指令少, 容易记忆, 计算机继续升位已有的操作符意义不变; 数据的具体类型依赖数据值的形式, 因此汇编器不能实现类型检查, 容易出错
* _AT&T_ 语法指令多, 难记忆, 计算机升位后现有操作符对应的字长可能改变; 汇编器可以实现类型检查