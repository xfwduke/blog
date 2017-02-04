---
layout: post
title: "找最大值"
date: 2014-12-05 21:30:12 +0800
comments: true
categories:
  - 汇编
tags:
  - Programming from the Ground Up Book
---

书上的第二个例子是在一堆数字里面找最大的一个, 这里的程序稍微改了下流程, 单基本还是差不多的. 另一个差别是, 最大值设定成了 _257_

这个程序的结果, 是用了 _sys___exit_ 调用返回的, 因为还没学到怎么把一个数字输出到屏幕上

<!--more-->


{% codeblock %}
.section .data
    data_items:
        .long 256,257,3,67,34,222,45,75,54,34,44,33,22,11,66,-1
.section .text
.globl _start


_start:
    movl $0, %edi
    movl $0, %eax
    movl $0, %ebx

start_loop:
    cmpl $0,%eax
    jl loop_exit
    movl data_items(,%edi,4), %eax
    incl %edi
    cmpl %ebx,%eax
    jle start_loop
    movl %eax,%ebx
    jmp start_loop

loop_exit:
    movl $1, %eax
    int $0x80
{% endcodeblock %}



# _sys\_exit_

|Name|%eax|%ebx|
|:---:|:---:|:---|
|sys_exit|1|int|

看系统调用的定义, 返回值是 _int_ 类型. 但根据 _Linux exit status_ 的定义, 最大只到 _255_

{% blockquote %}
3.7.5 Exit Status

The exit status of an executed command is the value returned by the waitpid system call or equivalent function. Exit statuses fall between 0 and 255, though, as explained below, the shell may use values above 125 specially. Exit statuses from shell builtins and compound commands are also limited to this range. Under certain circumstances, the shell will use special values to indicate specific failure modes.

For the shell’s purposes, a command which exits with a zero exit status has succeeded. A non-zero exit status indicates failure. This seemingly counter-intuitive scheme is used so there is one well-defined way to indicate success and a variety of ways to indicate various failure modes. When a command terminates on a fatal signal whose number is N, Bash uses the value 128+N as the exit status.

If a command is not found, the child process created to execute it returns a status of 127. If a command is found but is not executable, the return status is 126.

If a command fails because of an error during expansion or redirection, the exit status is greater than zero.

The exit status is used by the Bash conditional commands (see Conditional Constructs) and some of the list constructs (see Lists).

All of the Bash builtins return an exit status of zero if they succeed and a non-zero status on failure, so they may be used by the conditional and list constructs. All builtins return an exit status of 2 to indicate incorrect usage.
{% endblockquote %}

只用到了 _int_ 的低 _8_ 位, 所以上面代码的结果最终会是 _1_