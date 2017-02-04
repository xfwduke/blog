---
layout: post
title: "真实的浮点数"
date: 2015-03-27 21:26:36 +0800
comments: false
categories:
  - 汇编
tags:
  - Professional Assembly Language
---

在认真学习浮点操作之前, 手欠的去看了下 _C_ 语义汇编出来的代码, 想看看真实情况下编译器是怎么把高级语言的浮点操作翻译成汇编语言的

<!--more-->

{% codeblock lang:c %}
#include <stdio.h>
int main()
{
    float a = 123.7;
    float b = 332.16;
    float c = a + b;
    printf("%f\n", c);
    return 0;
}
{% endcodeblock %}

## 追踪浮点在哪里
上面这段代码用 _gcc_ 编译成汇编, 是这个样子

{% codeblock %}
        .file   "test.c"
        .section        .rodata
.LC2:
        .string "%f\n"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        pushl   %ebp
        .cfi_def_cfa_offset 8
        .cfi_offset 5, -8
        movl    %esp, %ebp
        .cfi_def_cfa_register 5
        andl    $-16, %esp
        subl    $32, %esp
        movl    .LC0, %eax
        movl    %eax, 28(%esp)
        movl    .LC1, %eax
        movl    %eax, 24(%esp)
        flds    28(%esp)
        fadds   24(%esp)
        fstps   20(%esp)
        flds    20(%esp)
        fstpl   4(%esp)
        movl    $.LC2, (%esp)
        call    printf
        movl    $0, %eax
        leave
        .cfi_restore 5
        .cfi_def_cfa 4, 4
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .section        .rodata
        .align 4
.LC0:
        .long   1123509862
        .align 4
.LC1:
        .long   1134957691
        .ident  "GCC: (Debian 4.7.2-5) 4.7.2"
        .section        .note.GNU-stack,"",@progbits	
{% endcodeblock %}第一次看到这段汇编, 完全崩溃了, 因为在代码里面根本找不到 _C_ 代码中定义的浮点数值

尝试在 _gdb_ 中反编译单步

{% codeblock lang:bash %}
(gdb) b 4
Breakpoint 1 at 0x8048425: file test.c, line 4.
(gdb) r
Starting program: /home/xfwduke/ASM/test

Breakpoint 1, main () at test.c:4
4	    float a = 123.7;
(gdb) disassemble
Dump of assembler code for function main:
   0x0804841c <+0>:	push   %ebp
   0x0804841d <+1>:	mov    %esp,%ebp
   0x0804841f <+3>:	and    $0xfffffff0,%esp
   0x08048422 <+6>:	sub    $0x20,%esp
=> 0x08048425 <+9>:	mov    0x80484f4,%eax
   0x0804842a <+14>:	mov    %eax,0x1c(%esp)
   0x0804842e <+18>:	mov    0x80484f8,%eax
   0x08048433 <+23>:	mov    %eax,0x18(%esp)
   0x08048437 <+27>:	flds   0x1c(%esp)
   0x0804843b <+31>:	fadds  0x18(%esp)
   0x0804843f <+35>:	fstps  0x14(%esp)
   0x08048443 <+39>:	flds   0x14(%esp)
   0x08048447 <+43>:	fstpl  0x4(%esp)
   0x0804844b <+47>:	movl   $0x80484f0,(%esp)
   0x08048452 <+54>:	call   0x8048300 <printf@plt>
   0x08048457 <+59>:	mov    $0x0,%eax
   0x0804845c <+64>:	leave
   0x0804845d <+65>:	ret
End of assembler dump.
(gdb) x 0x80484f4
0x80484f4:	0x42f76666
(gdb) x /f 0x80484f4
0x80484f4:	123.699997
(gdb) x /d 0x80484f4
0x80484f4:	1123509862
{% endcodeblock %}

从 _gdb pc_ 所指的位置, 可以看到 _C_ 代码第 _4_ 行和汇编代码 第 _18_ 行是对应的

汇编第 _18_ 行把标签 _.LC0_ 的值 _1123509862_ 载入 _%eax_ , 而程序执行时 _.LC0_ 的内存地址是 _0x80484f4_ , 用 _gdb x_ 命令查看对应内存地址的值, 可以看到按单精度浮点解释的话值是 _123.699997_ , 按整数解释的话是 _1123509862_ —— 参考 {% post_link fu-dian-shu-fan-wei 浮点数范围 %} 把对应二进制转换为单精度浮点, 确实是这个值

## 整型是什么样

{% codeblock lang:c %}
#include <stdio.h>
int main()
{
    int a = 13;
    int b = 29;
    int c = a + b;
    return 0;
}
{% endcodeblock %}
	
{% codeblock %}
        .file   "test.c"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        pushl   %ebp
        .cfi_def_cfa_offset 8
        .cfi_offset 5, -8
        movl    %esp, %ebp
        .cfi_def_cfa_register 5
        subl    $16, %esp
        movl    $13, -4(%ebp)
        movl    $29, -8(%ebp)
        movl    -8(%ebp), %eax
        movl    -4(%ebp), %edx
        addl    %edx, %eax
        movl    %eax, -12(%ebp)
        movl    $0, %eax
        leave
        .cfi_restore 5
        .cfi_def_cfa 4, 4
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .ident  "GCC: (Debian 4.7.2-5) 4.7.2"
        .section        .note.GNU-stack,"",@progbits
{% endcodeblock %}

## 不一样
从上面的分析可以看到, _gcc_ 在处理浮点数和整数的初始化时方法是不一样的. 这样做的原因现在还不了解, 等到后面明白了再来补全吧
