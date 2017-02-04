---
layout: post
title: "标签和立即数和 GDB"
date: 2014-12-08 21:30:34 +0800
comments: false
categories: 
  - 汇编
tags: 
  - Professional Assembly Language
---

汇编的标签, 立即数再加上 _gdb_ 会出现一些复杂的组合, 为了不混乱必须把相关的概念弄清楚

<!--more-->

### 标签

{% codeblock %}
.section .data
some_number:
	.long 0x1020304
{% endcodeblock %}

这段代码通常的解释是 : 在数据段定义了一个 _long_ 类型的数 _some_number_ , 值是 _0x1020304_ —— 这个值是专门设计的, 方便 _gdb_ 查看 —— _some_number_ 就是一个标签

计算机中数据都必须存放在内存中, 要访问数据就必须知道对应的内存地址, 所以这个定义更详细的含义是

* _some_number_ 表示内存中一个连续的 _4 Bytes_ 块
* _some_number_ 的值是这个内存块低位字节的地址
* 这个内存块中的值是 _0x1020304_

### 标签和 _gdb_

```bash
(gdb) p/x some_number
$3 = 0x1020304
(gdb) x some_number
0x1020304:	Cannot access memory at address 0x1020304
```

`p/x some_number` 直接显示了 _0x1020304_ , 用 _x_ 指令访问 _some_number_ 被告知内存地址 _0x1020304_ 无法访问

```bash
(gdb) p &some_number
$1 = (<data variable, no debug info> *) 0x8049080
(gdb) x/4bx &some_number
0x8049080 <some_number>:	0x04	0x03	0x02	0x01
```

`p &some_number` 显示了一个 _32_ 位数字, 用 _x_ 指令访问其指向的 _4_ 字节, 得到了 _0x1020304_ (考虑 _little end_ )

所以在 _gdb_ 中, 标签名 _some_number_ 代表了变量的值, 而 _&some_number_ 才是变量的地址

### 标签和立即数

用程序处理数据, 除了处理数据的值总会有需求要处理数据的地址. 在汇编中, 可以用 `$some_number` 的形式来取得对应的地址 —— `$` 是汇编中的立即数前导符

_movx_ 指令总会把 _source_ 指向的内存地址的值传送到 _destination_ (当 _source_ 是一个内存地址时) —— 当 _source_ 不是寄存器, 也不用 _$_ 前导时, _mov_ 总会把其当做一个内存地址

{% codeblock %}
.section .data
.section .text
.global _start

_start:
    movl 1, %eax
{% endcodeblock %}

上面这段代码可以正常编译连接, 但指向的时候会出现 _Segment Fault_ —— 因为会尝试把内存地址为 _1_ 的值取出来 —— 这是不允许的

{% codeblock %}
.section .data
some_number:
    .long 0x1020304
.section .text
.global _start

_start:
    movl some_number, %eax
{% endcodeblock %}

在 _gdb_ 中单步执行完 _movl_ 指令后, 查看寄存器 _eax_ 的值

```bash
(gdb) p/x $eax
$2 = 0x1020304
```

修改下上面的代码, 在 _some_number_ 前加一个 _$_ 前导符

{% codeblock %}
.section .data
some_number:
    .long 0x1020304
.section .text
.global _start

_start:
    movl $some_number, %eax	#modify here
{% endcodeblock %}

再在 _gdb_ 中跟踪下代码执行情况

```bash
(gdb) p &some_number
$1 = (<data variable, no debug info> *) 0x804907c
(gdb) p/x $eax
$3 = 0x804907c
(gdb) x/4bx $eax
0x804907c <some_number>:	0x04	0x03	0x02	0x01
```

此时 _eax_ 的值确实是 _0x1020304_ 的对应的内存地址了 —— 这是因为 _$_ 前导符把 _some_number_ 的值 _0x804907c_ 当做立即数传送给了 _eax_ , 而不是去这个位置取 _4 Bytes_ 传送 —— `立即` 的意思就是 __本地搞定__

_$_ 这个东西作用在标签上的行为, 会很容易让人联想到 _c_ 的指针行为

```c
int a = 1234;
```

这里 _a_ 表示 _1234_, _&a_ 表示 _1234_ 所在内存的地址 (倒是和 _gdb_ 中一样的) —— _$_ 就是汇编中取地址的操作符, 这样理解不是比搞出个`立即数`的概念更简单? —— 实际不能这样理解

看下面这段代码

{% codeblock %}
.section .data
.section .text
.global _start

_start:
    movl $1, %eax
{% endcodeblock %}

前面已经知道, `movl 1, %eax` 会因为寻址到内存 _1_ 位置导致 _Segment Fault_

而 `movl $1, %eax` 则会直接把数值 _1_ 传送到 _eax_

所以, 再汇编中 _$_ 前导符有非常明确的含义 __不做寻址, 直接操作其后的数值__

### 结论

前面那么多都是为了梳理思路和概念, 才能对结论的印象更深

对于在数据段中定义的元素

* 要取得元素的值
    * 在汇编中直接引用标签
    * 在 _gdb_ 中直接引用标签
* 要取得元素的地址
    * 在汇编中添加 _$_ 前导
    * 在 _gdb_ 中添加 _&_ 前导
