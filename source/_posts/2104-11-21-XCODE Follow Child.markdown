---
layout: post
title: "Xcode Follow Child"
date: 2014-11-21 21:41:09 +0800
comments: false
categories:
  - C/C++
tags:
  - Advanced Programming in the UNIX Environment
---




GDB 调试多进程代码可以用 `set follow-fork-mode child` 跟踪到子进程中去. 但是在 LLDB 中还没有对应的方法. 系统更新到 OS X 10.10 后 GDB 没有发布对应的版本, 造成现在调试多进程代码非常的麻烦  

在 Stack Overflow 上找到一篇不错的文章, 自己试了下. 发现其实可以比文章中的方法更加简单  
<!--more-->

## 核心

	SIGSTOP

> 这是一个作业控制信号, 用于停止一个进程, 不能被捕捉或者忽略

## 代码实例

```c
#include <stdio.h>
#include "apue.h"
int main(int argc, const char * argv[])
{
    pid_t pid;
    if ((pid = fork()) < 0) {
        err_sys("fork error");
    } else if (pid > 0) {
        printf("in parent\n");
    } else {
        kill(0, SIGSTOP);		//IMPORTANT
        printf("in child\n");
        exit(0);
    }
    return 0;
}
```

在刚进入子进程时, 发送了一个 SIGSTOP 信号 : 这是调试子进程的关键

## XCODE 操作

{% img /images/XCODE_follow_child_1.png 300 %}


在代码中加了个断点, 调试起来会比较方便  

接下来直接执行就好了. 执行后程序会直接返回 : 因为父进程随便输出一行就结束了, 但是由于在子进程加了停止信号, 所以子进程是还在的  

接着选择 Debug 菜单下的 Attach to Process 项

{% img /images/XCODE_follow_child_2.jpg 300 %}


会看到一个和当前工程同名的进程, 这个就是停止的子进程

{% img /images/XCODE_follow_child_3.jpg 300 %}


单击选中即可, 程序就会自动在前面的断点处暂停. 接着想怎么调试都可以

[来源:Stack Overflow](http://stackoverflow.com/questions/20161144/command-line-application-how-to-attach-a-child-process-to-xcode-debugger)
