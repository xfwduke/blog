---
layout: post
title: "冒泡排序"
date: 2014-12-09 21:38:17 +0800
comments: false
categories: 
  - 汇编
tags: 
  - Professional Assembly Language
---

已经学了数据传送, 对比, 跳转, 交换 等指令, 用学过的来实现一个冒泡排序

代码写的和书上的有些不同, 但是参考了书上的一些东西, 如

1. 把 _%esi_ 用作指向元素的指针
2. _xchg_ 的用法
3. 用 _skip\_xchg_ 标签来实现控制是否执行交换元素的值的操作

<!--more-->

### 算法分析

从冒泡排序的算法可以知道它包含两层循环

* 内层循环负责把最大值下沉到序列末端
* 外层循环负责在未下沉序列中迭代调用内层循环

所以算法的大体结构是

```
	迭代循环
		下沉逻辑
```

### 单次下沉

把序列想象成一个从左到右的序列, 每次循环都是从左向右处理每个元素

排序最核心的, 就是把最大值下沉到序列末端

对于包含 _N_ 个数的序列, 依次比较相邻两个数的大小, 如果右边的值比左边的大则交换其值, 这样遍历整个序列后最大值就在序列最右端 ( 末端 )

{% codeblock 单次下沉 %}
.section .data
list:
    .long 22, 17, 5, 8, 93, 145, 62, 555, 908, 0, 7
CNT:
    .long 11

.section .text
.globl _start

_start:
    movl $list, %edi
    movl $1, %ebx
_single_bubble:
    movl (%edi), %eax
    cmpl %eax, 4(%edi)
    jge _skip_xchg
    xchg %eax, 4(%edi)
    movl %eax, (%edi)
_skip_xchg:
    addl $4, %edi
    incl %ebx
    cmpl CNT, %ebx
    jl _single_bubble

_exit:
    movl $1, %eax
    movl $0, %ebx
    int $0x80
{% endcodeblock %}

* 一个有 _N_ 个元素的序列
* _%edi_ 是指向序列中元素的指针 —— 当前元素
* _%ebx_ 是当前处理的元素 ( _%edi_ 指向 ) 的下标
* 比较 _%edi_ 指向的元素和下一个元素的值 : _line 14 ~ 15_
    * 如果左边 ( 当前元素 ) 的值大于等于右边边的值大于等于右边ine 16 ~ 18
* 当 _%ebx_
    * _== N - 1_ 时, 所有的元素都参与过比较了, 退出循环
    * _< N - 1_ 时, 向右移动 _%edi_ 并将 _%ebx_ 自曾 _1_ , 继续循环


### 外层迭代

对于包含 _N_ 个元素的序列, 执行完一次下沉操作后, 最右边是序列的最大值, 左边的 _1 ~ N - 1_ 个元素依旧是无序的, 需要在其上再次做下沉操作 —— 外层迭代主要就是负责这个工作

补全前面的代码, 加入迭代逻辑

{% codeblock 外层迭代 %}
.section .data
list:
    .long 22, 17, 5, 8, 93, 145, 62, 555, 908, 0, 7
CNT:
    .long 11

.section .text
.globl _start

_start:
    movl CNT, %ecx
_outer:
    movl $list, %edi
    movl $1, %ebx
_single_bubble:
    movl (%edi), %eax
    cmpl %eax, 4(%edi)
    jge _skip_xchg
    xchg %eax, 4(%edi)
    movl %eax, (%edi)
_skip_xchg:
    addl $4, %edi
    incl %ebx
    cmpl %ecx, %ebx
    jl _single_bubble
    dec %ecx
    cmpl $1, %ecx
    jg _outer

_exit:
    movl $1, %eax
    movl $0, %ebx
    int $0x80
{% endcodeblock %}

* _%ecx_ 表示需要做下沉操作的序列的最大下标, 对应的序列是 _1 ~ %ecx_
* 每完成一次下沉 ( _line 15 ~ 25_ ), 就把 _%ecx_ 自减 _1_
    * 如果 _%ecx > 1_, 则继续在 _1~ %ecx_ 上做下沉操作 : _line 26 ~ 27_
    * 如果 _%exc == 1_, 则下沉序列中只有 _1_ 个元素, 排序完成, 退出循环 : _line 28_

程序中的下标是从 _1_ 开始, 所以有 _N_ 个元素的序列, 下标是 _[ 1, N ]_ —— 不同于 _c_ 常用的 _[ 0, N )_ —— 这样做写程序比较方便