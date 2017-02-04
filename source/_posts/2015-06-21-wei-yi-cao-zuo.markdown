---
layout: post
title: "位移操作"
date: 2015-06-21 15:51:58 +0800
comments: false
categories:
  - 汇编
---

位移操作主要包含 _3_ 个部分[^1]

[^1]:《Intel 64 and IA-32 Architectures Software Developer's Manual Vomume 2》

* _Shift bits_
* _Double-shift bits ( move them between operands )_
* _Rotate bits_

<!--more-->

## _Shift Instructions_

* 左位移
    * `SAL` —— 算术左位移 ( _shift arithmetic left_ )
    * `SHL` —— 逻辑左位移 ( _shift logical left_ )
* 右位移
    * `SAR` —— 算术右位移 ( _shift arithmetic right_ )
    * `SHR` —— 逻辑右位移 ( _shift logical right_ )


* 操作数 ( _operand_ )可以是 _8 / 16 / 32 / 64 bit_ 的寄存器或内存
* _shifter_ 可以是 _1 , CL_ 或 _8 bit_ 立即数

所以有 _3_ 种形式[^1] ( 以 _SAL_ 为例 )

{% codeblock %}
SHL r/m, 1
SHL r/m, CL
SHL r/m, imm8
{% endcodeblock %}
	
### _AT&T_

在 _AT&T_ 语法中, _shifter_ 是在 _operand_ 之前, 即 `SHL shifter, operand`

同时还有个特殊的地方 —— _AT&T_ 把 `SHL 1, r/m` 的形式简化为一个单操作数指令 `SHL r/m` —— 这个在 _Intel_ 语法中是非法的 ( _nasm_ 汇编报错)


### _CF_


无论是左位移还是右位移, 都会把移除操作数的位的值存入 _CF_ , 如果 $shifter\gt1$ 则 _CF_ 保存的是被移除的最后一个位

这个特性结合左位移可以实现数值到二进制的转换 {% post_link convert-int-to-binary 十进制转换成二进制 %}

### 算术位移


先看 `SAR` , 一段简单的程序就可以解释

{% codeblock %}
segment .data

segment .bss

segment .text

global _start

_start:
    mov al, 0b10000000                                                                                                                                               
    sar al, 1
    shr al, 1

_exit:
    mov rax, 60
    mov rdi, 0
    syscall
{% endcodeblock %}
	
{% codeblock %}
$1 = 10000000		#init value
$2 = 11000000		#after sar al, 1
$3 = 01100000		#after shl al, 1
{% endcodeblock %}
	
从执行结果可以看到, `SAR` 执行后会将高位填充为原来的符号位, 而 `SHR` 则是填 _0_

所以 `SAL` 和 `SHL` 是一样的, 因为不存在填充高位

## _Double-Shift Instructions_


_SHLD_ 和 _SHRD_

基本形式是

{% codeblock %}
SHRD r/m, r, imm8
SHRD r/m, r, CL
{% endcodeblock %}
	
需要注意的是

* 指令包含 _3_ 操作数
* 寄存器和内存的字长最小只支持 _16 bit_
* 同一指令中寄存器或内存字长必须一致

指令的作用是 : 从源 _r_ 中左/右移 _imm8/CL_ 位到 _r/m_ 中

所以换一种形式表达是 `SHRD destination, source, shifter`

而在 _AT&T_ 中的形式是 `SHRD shifter, source, destination`

## 其他的 _shift_ 指令

在文档[^1]中发现还有其他的位移指令, 作用都比较特殊, 稍微提一下

_SARX , SHLX , SHRX_

这一组指令更加特殊, 文档中的描述是 _Shift Without Affecting Flags_, 也是一组有 _3_ 个操作数的指令. 同时只有在支持 _BMI2_ [^2] 指令集的 _CPU_ 上才有效

[^2]:[Bit Manipulation Instruction Sets](https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets)

## _Rotate Instructions_

循环位移指令包含 _2_ 组

* 普通的循环位移
    * _ROL : rotate left_
    * _ROR : rotate right_
* 带进位的循环位移
    * _RCL : rotate through carry left_
    * _RCR : rotate through carry right_

循环位移指令有两个操作数, 在 _Intel_ 语法下的基本形式是

{% codeblock %}
RCL r/m, shifter
{% endcodeblock %}

目的操作数可以是 _8 ~ 64 bit_ 的寄存器或内存

普通的循环位移会做两件事情

1. 移除的位放到操作数的另外一头
2. 移除的位存入 _CF_

带进位的循环位移是将 _CF_ 附加到位移方向, 相当于给操作数扩展了 _1 bit_

* _RCL_ 将 _CF_ 附加到操作数的最左端
* _RCR_ 将 _CF_ 附加到操作数的最右端
