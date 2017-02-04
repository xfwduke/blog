---
layout: post
title: "再看 mov"
date: 2015-06-07 15:40:13 +0800
comments: false
categories:
  - 汇编
---

_mov_ 是汇编中最常用的操作之一, 在 {% post_link guan-yu-mov 关于 mov %} 中很初步的了解了 _32 bit AT&T_ 语法下的 _mov_ 操作

升级到 _64 bit_ 后 _mov_ 操作有了一些新的内容, 同时由于要一并学习 _Intel_ 语法, 所以 _mov_ 操作的内容又变得多了起来

<!--more-->

## _mov_ 指令

* _AT&T_ 语法的指令较多, 有 _b, w, l, q_ 四种后缀分别对应操作数为 _8bit, 16bit, 32bit, 64bit_ 的情况
* _Intel_ 只有一个指令 _mov_ , 但是可以和 {% post_link ptr-operator-yu-jie-qu PTR Operator %} 配合使用 : _byte ptr, word ptr, dword ptr, qword ptr_  —— 可以简写为 _byte, word, dword, qword_
* _mov_ 指令后的操作数, 源和目的操作数的位置在两种语法中是反的

## 源和目的的各种组合

* {% post_link moving-a-constant-into-a-register Moving a constant into a register %}
* {% post_link moving-values-from-memory-to-registers Moving values from memory to registers %}
* {% post_link moving-values-from-a-register-to-memory Moving values from a register to memory %}
* _Moving data from one register to another_

    * 寄存器字长必须相等
    * 源寄存器字长小于目的寄存器字长时, 需要做`位扩展操作`


