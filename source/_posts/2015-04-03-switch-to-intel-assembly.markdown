---
layout: post
title: "Switch to Intel Assembly"
date: 2015-04-03 19:53:07 +0800
comments: false
categories:
  - 汇编
---

用 _Intel Syntax_ 工作需要做如下设置

<!--more-->

## 汇编器

_nasm_

{% codeblock %}
nasm -g -f elf test.s -o test.o
nasm -g -f elf64 test.s -o test.o
{% endcodeblock %}
	
## 连接程序

连接程序并没有变化, 还是 _ld_

{% codeblock lang:bash %}
ld -dynamic-linker /lib/ld-linux.so.2 -lc test.o -o test
ld -dynamic-linker /lib64/ld-linux-x86-64.so.2 -lc test.o -o test
{% endcodeblock %}
	
## 调试器反汇编

{% codeblock lang:bash %}
$ cat ~/.gdbinit
set disassembly-flavor intel
{% endcodeblock %}

也可以直接使用命令行参数并添加一个 _alias_, 这样选择更加灵活

{% codeblock lang:bash %}
alias igdb="gdb --eval-command='set disassembly-flavor intel'"
{% endcodeblock %}## _gcc_ 汇编代码

{% codeblock lang:bash %}
gcc -S -masm=intel test.c
{% endcodeblock %}

## _objdump_ 反汇编

{% codeblock lang:bash %}
objdump -M intel -D test
{% endcodeblock %}
	
## _useful website_

[Linux Assembly](http://asm.sourceforge.net)

这个网站的 _tutorial_ 分类下提供一个 _AT%T_ 语法转换到 _Intel_ 语法的工具
	

