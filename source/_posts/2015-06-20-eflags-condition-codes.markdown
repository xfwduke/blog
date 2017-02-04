---
layout: post
title: "EFLAGS Condition Codes"
date: 2015-06-20 14:19:44 +0800
comments: false
categories:
  - 汇编
---

汇编中有非常多的条件指令, 如 _CMOVcc , FCMOVcc, Jcc, SETcc_ 等

这些指令都是参考 _EFLAGS_ 的 _CF OF SF ZF PF_ 状态位来决定如何执行动作[^1]

<!--more-->
	
_OR_|_AND_|_NOT_|_XOR_|
:----:|:----:|:----:|:----:
$\lor$  | $\land$  | $\lnot$  |$\oplus$  

  
## ___Conditions___ 

_Mnemonic(cc)_|_Condition Tested For_ |_Status Flags Setting_
:----:|:----:|:----:
_O_|_Overflow_|$OF=1$
_NO_|_No overflow_|$OF=0$
_B<br>NAE_|_Below<br>Neither above nor equal_|$CF=1$
_NB<br>AE_|_Not below<br>Above or equal_|$CF=0$
_E<br>Z_|_Equal<br>Zero_|$ZF=1$
_NE<br>NZ_|_Not equal<br>Not zero_|$ZF=0$
_BE<br>NA_|_Below or equal<br>Not above_|${CF}\lor{ZF}=1$
_NBE<br>A_|_Neither below not equal<br>Above_|${CF}\lor{ZF}=0$
_S_|_Sign_|$SF=1$
_NS_|_No Sign_|$SF=0$
_P<br>PE_|_Parity<br>Parity even_|$PF=1$
_NP<br>PO_|_No parity<br>Parity odd_|$PF=0$
_L<br>NGE_|_Less<br>Neither greater nor equal_|${SF}\oplus{OF}=1$
_NL<br>GE_|_Not less<br>Greater or equal_|${SF}\oplus{OF}=0$
_LE<br>NG_|_Less or equal<br>Not greater_|$({SF}\oplus{OF})\lor{ZF}=1$
_NLE<br>G_|_Neither less or equal<br>Greater_|$({SF}\oplus{OF})\lor{ZF}=0$

## 别名

从表格中可以看到部分 _Status Flags Setting_ 对应两个 _Mnemonic_ , 如 _E_ 和 _Z_

所以如 _JE_ 和 _JZ_ 实际是同一个指令的不同名字



## 无符号和带符号

表格中涉及到 __大小__ 比较的 _Mnemonic_ 有多处, 大致可以分为两类

* _above / below_ —— 用于无符号整数 ( _unsigned int_ )的比较
* _greater / less_ —— 用于带符号整数 ( _signed int_ )的比较



{% codeblock %}
.section .data

.section .bss

.section .text

.globl _start

_start:
    movb $0x40, %al
    movb $0xA0, %bl
    cmp %al, %bl

_exit:
    movq $60, %rax
    movq $0, %rdi
    syscall
{% endcodeblock %}
	
* _%al_ 的值是 _64_
* _%bl_ 的值, 看做无符号整型是 _160_ , 看做带符号整型是 _-96_
* 比较 _bl_ 大于还是小于 _al_

{% codeblock %}
(gdb) p $eflags
$4 = [ PF IF OF ]
{% endcodeblock %}
	
执行结果是 _OF_ 被置位 ( _PF_ 先不管)

所以 _EFLAGS Condition_ 参考的几个状态为的值分别是  

$OF=1 , CF=0 , SZ=0 , ZF=0$

* $ZF=0\Longrightarrow Not\ equal\ /\ Not\ zero$
* _above / below_ 类
    * $CF=0\Longrightarrow Not\ below\ /\ Above\ or\ equal$
    * $CF\lor{ZF}=0\Longrightarrow Neither\ below\ nor\ equal\ /\ Above$
* _greater / less_ 类
    * $SF\oplus OF=1\Longrightarrow {Less}\ /\ Neither\ greater\ not\ equal$
    * $(SF\oplus OF)\lor ZF)=1 \Longrightarrow Less\ or\ equal\ /\ Not\ greater$


[^1]:《ASM86 LANGUAGE REFERNCE MANUAL》 : B.1 CONDITION CODES
