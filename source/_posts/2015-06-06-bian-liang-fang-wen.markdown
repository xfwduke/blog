---
layout: post
title: "变量访问"
date: 2015-06-01 20:48:56 +0800
comments: false
categories:
  - 汇编
---

特指在 `data段` 定义的变量 

在汇编里面, `变量` 这个东西真的是个很容易把人弄迷糊的概念, 因为总是和 `内存` 掺和在一起

再加上同时在看 _AT&T_ 和 _Intel_ 两种语法, 就更加混乱了

* _AT&T_ 中有 `variable` 和 `$variable` 两种形式
* _Intel_ 中有 `variable` 和 `[variable]` 两种形式

<!--more-->

## _AT&T_

{% codeblock %}
anum:
    .int 0x11223344
{% endcodeblock %}

### _variable_

{% codeblock %}
.section .data
anum:
    .int 0x11223344

.section .bss

.section .text

.globl _start

_start:
    xor %rax, %rax
    movl anum, %eax

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}
	
{% codeblock lang:bash %}
(gdb) p /x $rax
$2 = 0x11223344
{% endcodeblock %}

通过直接访问变量名获得了变量的值
	
### _$variable_

{% codeblock %}
.section .data
anum:
    .int 0x11223344

.section .bss

.section .text

.globl _start

_start:
    xor %rax, %rax
    movl $anum, %eax

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}
	
{% codeblock lang:bash %}
(gdb) p /x $rax
$2 = 0x6000c8
(gdb) x /4bx 0x6000c8
0x6000c8:       0x44    0x33    0x22    0x11
{% endcodeblock %}
	
可以看到, 在变量名前添加 _$_ 后取到的是变量值所在的内存地址

## _Intel_

{% codeblock %}
anum dd 0x11223344
{% endcodeblock %}


### _variable_


{% codeblock %}
segment .data
anum dd 0x11223344

segment .bss

segment .text

global _start

_start:
    xor rax, rax
    mov eax, anum

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock %}
(gdb) p /x $rax
$2 = 0x6000d0
(gdb) x /4bx 0x6000d0
0x6000d0 <anum>:        0x44    0x33    0x22    0x11
{% endcodeblock %}

在 _Intel_ 语法中, 直接访问变量名得到的是内存地址
	
### \[_variable_\]

用 `[variale]` 形式得到的是变量的值