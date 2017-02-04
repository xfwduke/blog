---
layout: post
title: "Xcode debug sleep in parent"
date: 2014-11-21 21:41:05 +0800
comments: false
categories:
  - C/C++
tags:
  - Advanced Programming in the UNIX Environment
---



在写多进程代码时, 有时候为了方便查看父/子进程的执行序列, 会随手在代码中添加 sleep 调用. 但是这样的调用, 在 Xcode 中以 debug 方式执行程序时, 执行序列很可能是不正确的
<!--more-->

## 实例

{% codeblock lang:c %}
#include<stdio.h>
#include"apue.h"
#include<time.h>

int main() {
    pid_t pid;
    if((pid = fork()) < 0) {
        err_sys("fork error");
    } else if(pid > 0) {
        printf("%d : enter parent %d\n", (int)time(NULL), getpid());
        sleep(2);
        printf("%d : after sleep 2 %d\n", (int)time(NULL), getpid());
        sleep(3);
        printf("%d : after sleep 3 %d\n", (int)time(NULL), getpid());
    } else {
        printf("%d : enter child %d\n", (int)time(NULL), getpid());
    }
    return 0;
}
{% endcodeblock %}

这是一段非常简单的代码, 不执行也能想象出程序的结果

* 两句 enter 几乎在同一时间输出
* 父进程中剩下的两句分别在 2 秒和 3 秒后输出

当在 Xcode 中直接以 debug 方式执行时, 结果却是下面这样

```
1414914313 : enter parent 19071
1414914313 : enter child 19074
1414914313 : after sleep 2 19071
1414914316 : after sleep 3 19071
Program ended with exit code: 0
```

从结果可以看到, 有一个 sleep 没有正常执行 (在这个例子中是第一个 sleep(2) )

如果在子进程代码中故意加入特定的 sleep 调用, 还可以各种情况, 比如

* 第二个 sleep 完全没执行
* 两个 sleep 的其中一个执行了不足设定的时间

例如, 如果在子进程的 printf 语句后添加一行 sleep(1), 那输出结果会是

```bash
1414914595 : enter parent 19108
1414914595 : enter child 19111
1414914596 : after sleep 2 19108
1414914599 : after sleep 3 19108
Program ended with exit code: 0
```

## 原因分析
* sleep
* SIGCHLD

### sleep

这里的关键就是 sleep 函数, 在我们的印象中, sleep 总会让程序暂停我们设定的时间, 但是查看 manpage 会发现并非如此 (man 3 sleep)

```
     unsigned int
     sleep(unsigned int seconds);

DESCRIPTION
     The sleep() function suspends execution of the calling thread until either seconds seconds have elapsed or a signal is delivered to the thread and its action is
     to invoke a signal-catching function or to terminate the thread or process.  System activity may lengthen the sleep by an indeterminate amount.

     This function is implemented using nanosleep(2) by pausing for seconds seconds or until a signal occurs.  Consequently, in this implementation, sleeping has no
     effect on the state of process timers, and there is no special handling for SIGALRM.

RETURN VALUES
     If the sleep() function returns because the requested time has elapsed, the value returned will be zero.  If the sleep() function returns due to the delivery of
     a signal, the value returned will be the unslept amount (the requested time minus the time actually slept) in seconds.
```

从 manpage 可以看到几点容易被忽略的

1. sleep 并不总会让程序暂停设定的时间, 当进程接收到某个信号后, sleep 会直接返回
2. sleep 是有返回值的
    * 当 sleep 让程序暂停了整个设定时间, 返回值是 0
    * 当因为某种信号导致 sleep 提前结束, 返回值是剩余的时间

### SIGCHLD

前面已经知道了, sleep 在某些情况写确实会提前返回. 在本文的例子中, 正是因为子进程结束后向父进程发送的通知信号 SIGCHLD 导致的

下面是 Unix/Linux 系统中的一些默认约定

* 子进程结束时, 总会向父进程发送 SIGCHLD 信号
* 父进程默认情况下的 SIGCHLD 处理策略是 SIG_DFL

问题就在这个 SIG_DFL 了, Xcode 在 debug 模式执行程序时, 实际是调用的 lldb  

以下是推测
  
__lldb 会修改 SIGCHLD 的 SIG_DFL 策略, 不再忽略这个信号__

#### 验证一

在执行 fork 后, 马上在父进程强制设定忽略 SIGSHLD 信号

```
signal(SIGCHLD, SIG_IGN);
```

这样处理之后, 会发现 Xcode 的 debug 模式可以得到期望的输出结果

#### 验证二

修改 Xcode 执行程序的 schema 配置为 release 模式, 也可以得到期望的输出结果

#### 验证三

在命令行直接调用 clang 编译代码, 同样可以得到期望的输出结果

#### 验证四

在验证三中得到的可执行程序, 用 lldb 加载后执行, 输出结果异常

#### 验证五

在 Linux 环境下, 用 gdb 加载编译结果执行, 输出结果正常

## 总结

* 从上面的实验可以看到, lldb 貌似真的修改了某些信号的默认处理逻辑, 原因未知
* 当调试代码发现结果偏离预期, 但检查很多次代码后还是不能发现异常时, 是可以怀疑工具本身问题的

## 参考

[llvm 项目提交日志](http://lists.cs.uiuc.edu/pipermail/lldb-commits/Week-of-Mon-20131028/009894.html) : 有人提交了一个修改, 是针对 lldb 的信号处理策略的, 相关行刚好包括 SIGCHLD 信号  
[Stack Overflow](http://stackoverflow.com/questions/12593449/whats-wrong-with-lldb-gdb-ignoring-first-sleep-statement-in-os-x) : Stack Overflow 上同样主题的帖子