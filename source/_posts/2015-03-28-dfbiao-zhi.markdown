---
layout: post
title: "DF 标志"
date: 2015-03-28 21:44:14 +0800
comments: false
categories:
  - 汇编
tags:
  - Professional Assembly Language
---

在 {% post_link trans-string 字符串传送 %} 中, 使用 _movsb_ 指令复制了一个长度为 _23_ 的字符串. 在示例代码中可以看到 _esi_ 和 _edi_ 没有显示的自增操作, 却可以在循环中自动的每次加 _1_.

每次执行 _movs_ 指令时, 数据传送后 _esi_ 和 _edi_ 的值会自动改变, 为另一次传送做准备, 默认情况下

* 执行 _movsb_ 后 _esi_ 和 _edi_ 的值都增加 _1_
* 执行 _movsw_ 后 _esi_ 和 _edi_ 的值都增加 _2_
* 执行 _movsl_ 后 _esi_ 和 _edi_ 的值都增加 _4_

实际上 _esi_ 和 _edi_ 是可以向两个方向变动的, 即可以增加也可以减少, 而这一行为是受 _eflags_ 的 _df_ 标志影响

* _df_ 的值如果为 _0_ 则 _esi_ 和 _edi_ 递增
* _df_ 的值如果为 _1_ 则 _esi_ 和 _edi_ 递减

有两个指令专门用于操作 _df_ 标志的值

* _cld_ 将 _df_ 标志置 _0_
* _std_ 将 _df_ 标志置 _1_

通常情况下 _df_ 的默认值是 _0_ , 而我们常规的思维也更习惯 _df_ 为 _0_ 时的逻辑. 但是为了避免意外情况, 建议总是显示的调用 _cld_ 指令.

<!--more-->

## $DF=1$

{% codeblock %}
.section .data
my_str:
    .ascii "This is a test string.\n"

.section .bss
    .lcomm output, 23

.section .text

.globl _start

_start:
    movl $my_str+22, %esi
    movl $output+22, %edi
    std
    movsw
    movsw
    movsw

_exit:
    pushl $0
    call exit
{% endcodeblock %}    
用一个简单的程序来看看 _df_ 为 _1_ 时的情况

{% codeblock lang:bash %}
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	10 '\n'
{% endcodeblock %}
	
{% codeblock lang:bash %}
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	103 'g'	46 '.'	10 '\n'
{% endcodeblock %}
	
{% codeblock lang:bash %}
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	105 'i'	110 'n'	103 'g'	46 '.'	10 '\n'
{% endcodeblock %}

每次执行 _movsw_ 后都打印 _output_ 的值, 可以看到内容确实是从末尾开始填充的, 但奇怪的是 _3_ 次 _movsw_ 只复制了 _5_ 字节, 其中第 _1_ 次 _movsw_ 只复制了最末尾的换行符
	
修改下代码,  换成 _movsl_ 看看

{% codeblock %}
.section .data
my_str:
    .ascii "This is a test string.\n"

.section .bss
    .lcomm output, 23

.section .text

.globl _start

_start:
    movl $my_str+22, %esi
    movl $output+22, %edi
    std
    movsl
    movsl

_exit:
    pushl $0
    call exit
{% endcodeblock %}
	
{% codeblock lang:bash %}
(gdb) x /23c &output
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	10 '\n'
(gdb) n
20	    pushl $0
(gdb) x /23c &output
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	105 'i'	110 'n'	103 'g'	46 '.'	10 '\n'
{% endcodeblock %}
	
从 _gdb_ 结果可以看到, 第一次 _movsl_ 还是只复制了一个末尾的换行符

再把代码修改下[^1]

[^1]:汇编语言程序设计-chapter 10-P218

{% codeblock %}
.section .data
my_str:
    .ascii "This is a test string.\n"

.section .bss
    .lcomm output, 23

.section .text

.globl _start

_start:
    movl $my_str+22, %esi
    movl $output+22, %edi
    std
    movsb
    movsw
    movsl

_exit:
    pushl $0
    call exit
{% endcodeblock %}
	
{% codeblock lang:bash %}
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	10 '\n'
{% endcodeblock %}
	
执行 _movsb_ 后, 复制了末尾的换行符, 这是符合 _movsb_ 一次一字节的语义的

{% codeblock lang:bash %}
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	46 '.'	10 '\n'
{% endcodeblock %}
	
接着执行 _movsw_ , 奇怪的事情就发生了 —— 总共只有 _2_ 字节被复制到 _output_ 末尾

{% codeblock lang:bash %}
0x80492b0 <output>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492b8 <output+8>:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0x80492c0 <output+16>:	0 '\000'	0 '\000'	0 '\000'	110 'n'	103 'g'	46 '.'	10 '\n'
{% endcodeblock %}
	
再接着执行 _movsl_ , 更奇怪了 —— 总共只有 _4_ 字节被复制到 _output_ 末尾

这一系列奇怪的行为, 其实是和 _movs_ 指令的一个特性有关 —— _movs_ 指令总是从 _esi_ 存储的内存位置向递增方向读取特定长度的字节序, 复制到 _edi_ 存储的内存位置, 并按递增方向填充 —— 即使将 _df_ 置 _1_ 也不会改变这个特性

在这个前提下, 再来分析下上面的代码

* _df_ 被置 _1_
* _16_ 行执行前, _esi edi_ 分别指向源和目的的末尾
* _16_ 行的 _movsb_ 从源末尾开始读取 _1_ 字节, 即换行符复制到目的, 然后 _esi edi_ 各自减 _1_ 分别指向源和目的的倒数第 _2_ 字节
* _17_ 行的 _movsw_ 从源的倒数第 _2_ 字节开始读取 _2_ 字节, 即 . 和换行符复制到目的倒数第 _2_ 字节, 并向递增方向填充 —— 所以在 _gdb_ 中看到在 _movsw_ 执行后, 目的字符串末尾只有 _2_ 个字节 —— 然后 _esi edi_ 各自减 _2_ 分别指向源和目的的倒数第 _4_ 字节
* _18_ 行的分析同上

这也就解释了为什么连续 _n_ 个 _movsw_ 或 _movsl_ , 第 _1_ 条指令总是只复制最末尾的 _1_ 个字符 —— 因为无论是读取源还是写目的, 总是递增方向

总结下来就是

* _df_ 的状态, 只会影响 _movs_ 指令执行后 _esi edi_ 如何变化
* _movs_ 指令本身的寻址, 总是以 _esi edi_ 为起点向递增方向(后)寻址

鉴于当 _df_ 为 _1_ 时会造成这样的混乱, 实在不明白这种行为的好处, 但是既然存在, 总是要了解下的