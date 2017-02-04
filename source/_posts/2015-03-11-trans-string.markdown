---
layout: post
title: "传送字符串" 
date: 2015-03-27 23:26:11 +0800
comments: false
categories:
  - 汇编
tags:
  - Professional Assembly Language
---

{% codeblock %}
.section .data

my_str:
    .ascii "This is a test string.\n"   #length is 23

.section .bss

.section .text

.globl _start

_start:
    xor %eax, %eax
    movl $my_str, %eax
    movl $0x41424344, 3(%eax)

_exit:
    pushl $0
    call exit
{% endcodeblock %}

这段代码, 在执行了 `movl $my_str, %eax` 后发生了什么?

<!--more-->

在 _gdb_ 中调试看看, 将程序 _break_ 在 _15_ 行

```bash
(gdb) p &my_str
$5 = (<data variable, no debug info> *) 0x8049298
(gdb) x /23bc &my_str
0x8049298 <my_str>:	84 'T'	104 'h'	105 'i'	115 's'	32 ' '	105 'i'	115 's'	32 ' '
0x80492a0 <my_str+8>:	97 'a'	32 ' '	116 't'	101 'e'	115 's'	116 't'	32 ' '	115 's'
0x80492a8 <my_str+16>:	116 't'	114 'r'	105 'i'	110 'n'	103 'g'	46 '.'	10 '\n'
(gdb) p /x $eax
$7 = 0x8049298
```

从调试结果可以看到, _%eax_ 的值等于字符串 _my_str_ 的首地址

现在执行第 _15_ 行, 看看会是什么结果

```bash
(gdb) p (char[23])my_str
$8 = "ThiDCBA a test string.\n"
```

可以看到, 字符串的值被修改了 —— 这翻译成 _C_ 语言的说法就是 : _%eax_ 和 _my_str_ 是指向同一个字符串的指针


所以用 _mov_ 是不能传送字符串的



## _movs_ 指令

_movs_ 系列指令的作用是把字符串从一个内存位置传送到另一个内存位置 —— 存在两个字符串副本

* _movsb_ : 传送单一字节 ( _1_ 字节, _8bit_ )
* _movsw_ : 传送一个字 ( _2_ 字节, _16bit_ )
* _movsl_ : 传送一个双字 ( _4_ 字节, _32bit_ )

> _Intel_ 文档使用 _movsd_ 传送双字, _GNU_ 汇编器决定使用 _movsl_

_movs_ 指令使用隐含的源和目的操作数

* 隐含的源操作数是 _esi_ 寄存器, 指向源字符串的内存位置 —— _s_ 代表 _source_
* 隐含的目的操作数是 _edi_ 寄存器, 指向字符串要被复制到的目的内存位置 —— _d_ 代表 _destination_

{% codeblock %}
.section .data

my_str:
    .ascii "This is a test string.\n"   #length is 23

.section .bss
    .lcomm output, 23

.section .text

.globl _start

_start:
    movl $my_str, %esi
    movl $output, %edi
    xor %eax, %eax
    _begin:
        movsb
        incl %eax
        cmpl $23, %eax
        jl _begin
    movl $output, %eax
    movl $0x41424344, 3(%eax)
_exit:
    pushl $0
    call exit
{% endcodeblock %}

程序执行完第 _23_ 行的结果是什么呢? 在 _gdb_ 中调试下

```bash
(gdb) x /s &my_str
0x80492a8 <my_str>:	 "This is a test string.\n"
(gdb) x /s &output
0x80492c0 <output>:	 "ThiDCBA a test string.\n"
```

可以看到 _my_str_ 的值没有变化, 字符串被复制到了 _output_ 所在的内存位置, 内存中存在两份相互独立的副本