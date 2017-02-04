---
layout: post
title: "编译链接方法"
date: 2014-12-05 21:44:55 +0800
comments: false
categories:
  - 汇编
tags:
  - Programming from the Ground Up Book
---

最近在看 _Programming from the Ground Up_ , 在编译链接程序的时候需要注意

```bash
as --32 -gstabs+ source.s -o source.o
ld -m elf_i386 -o source source.o
```

