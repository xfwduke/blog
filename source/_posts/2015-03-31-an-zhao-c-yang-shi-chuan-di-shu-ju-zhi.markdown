---
layout: post
title: "按照 C 样式传递数据值"
date: 2015-03-31 20:49:22 +0800
comments: false
categories:
  - 汇编
tags:
  - Professional Assembly Language
---

在 {% post_link hui-bian-han-shu 汇编函数 %} 中介绍了关于汇编函数的基本概念, 其中有提到传送函数的参数和返回值均有多种方法

可以使用寄存器或全局变量来传送函数的参数和返回值, 但是这样做会极大的增加编码的难度

* 在实现由多个函数组合而成的复杂程序时, 需要仔细的跟踪每个函数如何影响这些全局对象
* 各函数的参数和返回值如何传送没有统一的标准, 强迫程序员编写大量的接口文档

结合栈 ( _stack_ ) 可以实现 _C_ 样式的值传递, 这更加适合函数式编程的开发

<!--more-->

## 回顾栈

* 程序启动后分配的内存空间末尾保留给栈
* 栈的起始是程序内存空间的最大值
* 当把数据填充入栈时, 栈顶地址变小
* _esp_ 用于指向栈顶

## 按 _C_ 样式传递参数

`C 样式要求参数存放到栈中的顺序和函数的原型中的顺序相反`

这是一个非常重要的规则 —— 不仅约定了用栈传递参数, 还约定了按什么样的顺序传递参数

### 参数逆序入栈

{% codeblock lang:c %}
#include <stdio.h>

void my_func(char * str, int a, int b)
{
    printf("%s, %d, %d\n", str, a, b);
}

int main()
{
    my_func("Hello world!", 13, 37);
    return 0;
}
{% endcodeblock %}

{% codeblock %}
...
.LC1:
        .string "Hello world!"
        .text
        .globl  main
        .type   main, @function
main:
.LFB1:
        .cfi_startproc
        pushl   %ebp
        .cfi_def_cfa_offset 8
        .cfi_offset 5, -8
        movl    %esp, %ebp
        .cfi_def_cfa_register 5
        andl    $-16, %esp
        subl    $16, %esp
        movl    $37, 8(%esp)
        movl    $13, 4(%esp)
        movl    $.LC1, (%esp)
        call    my_func
        movl    $0, %eax
        leave
        .cfi_restore 5
        .cfi_def_cfa 4, 4
        ret
        .cfi_endproc
...
{% endcodeblock %}

这是一段非常简单的 _c_ 代码, 同时截取了 _main_ 函数部分的汇编代码

先忽略汇编代码的其他部分, 只关注 _18 ~ 21_ 行

* _my\_func(char * str, int a, int b)_ 是函数原型
* 第 _21_ 行调用了 _my\_func_ 函数
* 第 _18 ~ 20_ 是将函数的参数入栈, 注意顺序刚好和原型相反
    * 首先将 _37_ 入栈, 对应参数 _int b_
    * 接着将 _13_ 入栈, 对应参数 _int a_
    * 最后将字符串 _"Hello world!"_ 入栈, 对应参数 _char * str_

当程序执行进入 _my\_func_ 后, 系统还会默认的将程序执行到的当前内存地址入栈用于函数结束后的返回, 所以在函数内部可以用 _esp_ 间接寻址的方式获得参数的值

{% codeblock %}
4(%esp)
8(%esp)
12(%esp)
{% endcodeblock %}
	
### _ebp_ 保留栈顶

前面提到进入函数后可以用 _esp_ 间接寻址来获得参数的值, 但是这样会带来一个问题 —— 栈操作都会影响 _esp_ 的值, 所以如果在函数内使用 _push pop_ 操作后, 就很难再用固定的 _esp_ 间接寻址方式来获得参数的值

为了解决这个问题, 常用的办法是一进入函数内部就马上将 _esp_ 的值传送到 _ebp_ 保存起来, 后续要获取参数值之间用 _ebp_ 做间接寻址即可, 任意的栈操作都不会影响

同时, 为了防止程序其他地方用到 _ebp_ 的值, 需要先将 _ebp_ 保存下来; 函数执行结束后, 在 _ret_ 前再加以恢复

{% codeblock %}
function:
    pushl %ebp
    movl %esp, %ebp
    ...
    ...
    movl %ebp, %esp
    popl %ebp
    ret
{% endcodeblock %}
	
代码的模板大概就是这个样子

* 第 _2_ 行将 _ebp_ 的原值先保存到栈中
* 第 _3_ 行将 _esp_ 的值, 也就是当前栈顶地址传送到 _ebp_
* 接着便是函数的代码, 可以任意操作栈
* 函数结束时先恢复栈顶值, 即删除掉函数代码中入栈的所有数据
* 从栈中恢复函数执行前 _ebp_ 的值

这样做带来的一个变化是 : 栈中多了一个 _ebp_ 原值, 所以为了获取参数值, _esp_ 间接寻址要从 _8(%esp)_ 开始了### 局部变量

严格来说局部变量并不属于函数值传递的范畴, 但由于实现技术上也是用到了栈, 所以一起来说

简单来说, 汇编函数的局部变量, 是入函数后在栈中直接扩展出 _N_ 个空位, 保留给局部变量使用

{% codeblock %}
function:
    pushl %ebp
    movl %esp, %ebp
    subl $N, %esp
    ...
    ...
{% endcodeblock %}
	
这样操作后栈顶部的 _N_ 字节保留给局部变量使用, 同时获取函数参数就要从 _(N + 4)(%esp)_ 开始了和函数参数类似, 获取和设置局部变量的值也是都使用 _ebp_ 间接寻址

由于在退出函数前还原了 _esp_ 的原值, 栈中的局部变量值就被删除了, 函数返回后从外部无法在访问这些值 —— 局部变量

## 在函数调用之后

{% codeblock %}
...
...
pushl %eax
pushl %ebx
call function
addl $8, %esp
...
...
{% endcodeblock %}
	
看这样一段示例 : 调用函数前将 _eax ebx_ 入栈作为参数, 所以在完成函数调用后, 需要还原 _esp_ 之前的值, 将栈空间还给系统以便后续使用

### _push / pop_ 和栈的间接寻址

从前面展示的代码中可以发现, 很多时候栈操作并没有使用 _push / pop_ 指令, 而是使用基于 _esp / ebp_ 的间接寻址方式 , 这样的选择的意义是[^1]

1. 当需要引用不在栈顶的数据时, 间接寻址更方便
2. 间接寻址速度更快
3. _ebp_ 间接寻址能保持各变量在栈中位置的稳定性, 不受其他的 _push / pop_ 影响, 减轻开发人员的负担
4. 早年的老 _CPU_ 还没有 _push / pop_ 指令

### 返回值

一般将函数返回值放在 _eax_ 中

[^1]:[stackoverflow : push local variables](http://stackoverflow.com/questions/4308021/push-local-variables)