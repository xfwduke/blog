---
layout: post
title: "开始位置标签"
date: 2014-12-06 23:09:30 +0800
comments: false
categories: 
  - 汇编
tags: 
  - Professional Assembly Language
---

`开始位置标签` 用于表明程序应该从哪里开始运行. 如果连接器找不到这个标签, 会报错
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

这段程序是书上的一个示例, 代码中使用 _\_start_ 作为 `开始位置标签` —— 这也是 _ld_ 的默认值 —— 如果想使用其他的标签, 则可以用 _-e_ 参数来指定, 如用 _main_

```bash
ld -o execute -e main source.o
```

用 _gcc_ 直接编译汇编代码时, 默认则是用 _main_ —— 这和 _c_ 的习惯一样