---
layout: post
title: "Hello World Again"
date: 2015-05-13 21:43:21 +0800
comments: false
categories: 
  - 汇编
---

_Hello World_ 很经典, 所以不得不来一个

<!--more-->

## _AT&T_

{% codeblock %}
.section .data
output:
    .asciz "Hello world!\n"

.section .bss

.section .text

.globl _start

_start:
    movq $output, %rdi
    call printf

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}

{% codeblock lang:bash %}
$ as hello.s -o hello.o
$ ld  -dynamic-linker /lib64/ld-linux-x86-64.so.2 -lc hello.o -o hello
{% endcodeblock %}

## _Intel_

{% codeblock %}
extern printf
segment .data
output: db  "Hello world!",10,0

segment .bss

segment .text

global _start

_start:
    mov rdi, output
    call printf

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock lang:bash %}
$ nasm -f elf64 hello.asm -o hello.o
$ ld  -dynamic-linker /lib64/ld-linux-x86-64.so.2 -lc hello.o -o hello
{% endcodeblock %}