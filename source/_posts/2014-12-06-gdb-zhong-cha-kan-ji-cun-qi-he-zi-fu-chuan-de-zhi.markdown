---
layout: post
title: "GDB 中查看寄存器和字符串的值"
date: 2014-12-06 23:41:52 +0800
comments: false
categories: 
  - 汇编
tags: 
  - Professional Assembly Language
---

在使用 _gdb_ 调试程序的时候, 经常需要查看寄存器和字符串的值


* 查看寄存器的值, 可以使用 _info register_
* 查看字符串的值, 可以使用 _x_ 来显示连续的内存区域


但是这些方法并不是非常直观

<!--more-->

{% codeblock %}
.section .data
output:
    .ascii "The processor Vendor ID is 'xxxxxxxxxxxx'\n"
.section .text
.globl _start

_start:
    movl $0, %eax
    cpuid

    movl $output, %edi
    movl %ebx, 28(%edi)
    movl %edx, 32(%edi)
    movl %ecx, 36(%edi)

    movl $4, %eax
    movl $1, %ebx
    movl $output, %ecx
    movl $42, %edx
    int $0x80

    movl $1, %eax
    movl $0, %ebx
    int $0x80
{% endcodeblock %}

还是以这个程序做例子, 执行结果是

```bash
$ ./cpuid
The processor Vendor ID is 'GenuineIntel'
```

和结果相关的三个寄存器的值分别是

```
%ebx : Genu
%edx : ineI
%ecx : ntel
```

用 _gdb_ 调试这个程序, 设置断点让程序执行完 _cpuid_ 指令

### 查看寄存器的值

```bash
(gdb) i r ebx edx ecx
ebx            0x756e6547		1970169159
edx            0x49656e69		1231384169
ecx            0x6c65746e		1818588270
```

_info register_ 的结果完全没有什么可读性 ------ 以 _ebx_ 为例, 把数值对照 _ASCII_ 码表做下转换, 倒是可以确定的确是对应的字符串

_print_ 也可以用来显示寄存器的值

```bash
(gdb) p $ebx
$1 = 1970169159
(gdb) p/c $ebx
$2 = 71 'G'
(gdb) p (char[])$ebx
$3 = "Genu"
```

由于事先就知道 _ebx_ 的值是个字符串, 所以 _p (char[])$ebx_ 的输出可读性最好

### 查看字符串的值

_output_ 

* 它是一个标签, 对应定义的字符串
* 它的值是字符串的首地址

```bash
(gdb) p/x output
$6 = 0x20656854
```

对照 _ASCII_ 码表, 可以知道上面的结果是 " ehT" ------ 为啥反过来了呢, 因为 _little end_

```bash
(gdb) x/42b &output
0x80490ac <output>:	84 'T'	104 'h'	101 'e'	32 ' '	112 'p'	114 'r'	111 'o'	99 'c'
0x80490b4 <output+8>:	101 'e'	115 's'	115 's'	111 'o'	114 'r'	32 ' '	86 'V'	101 'e'
0x80490bc <output+16>:	110 'n'	100 'd'	111 'o'	114 'r'	32 ' '	73 'I'	68 'D'	32 ' '
0x80490c4 <output+24>:	105 'i'	115 's'	32 ' '	39 '\''	120 'x'	120 'x'	120 'x'	120 'x'
0x80490cc <output+32>:	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'	120 'x'
0x80490d4 <output+40>:	39 '\''	10 '\n'
```

_x_ 指令的输出很详细, 但是如果只是想看看字符串的内容, 这样的方式有点多余

```bash
(gdb) printf "%s", &output
The processor Vendor ID is 'xxxxxxxxxxxx'
```

用 _printf_ 来查看, 输出就很人性化了

但是这种方法不能用来显示前面的 _ebx_, 因为 _ebx_ 的值不是一个字符串指针

---

```bash
.section .data
msg:
    .ascii "Hello World!\n"
.section .text
.global _start

_start:
    movl $msg, %eax
    movl msg, %ebx
    nop
```

先来看 `movl $msg, %eax`

语句把 _msg_ 当做一个 `立即数` 的值传入 _eax_ , 也就是字符串的首地址

```bash
(gdb) p &msg
$2 = (<data variable, no debug info> *) 0x8049080
(gdb) i r eax
eax            0x8049080	134516864
(gdb) printf "%s", $eax
Hello World!
```

再来看 `movl msg, %ebx`


语句把 _msg_ 指向的内存的值传入 _ebx_ , 由于 _ebx_ 的长度是 _32 bit (一个字长) _ , 所以 _ebx_ 的值是字符串的前 _4 Bytes_

```bash
(gdb) i r ebx
ebx            0x6c6c6548	 1819043144
(gdb) print (char[])$ebx
$3 = "Hell"
```
