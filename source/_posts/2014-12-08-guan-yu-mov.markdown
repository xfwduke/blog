---
layout: post
title: "关于 mov"
date: 2014-12-08 20:55:24 +0800
comments: false
categories: 
  - 汇编 
tags:
  - Professional Assembly Language
---

_mov_ 是通用的数据传送指令, 基本格式是

{% codeblock %}
movx source, destination
{% endcodeblock %}

根据数据元素的长度, 指令中的 _x_ 有几种选择

* _q_ 用于 _64_ 位的值
* _l_ 用于 _32_ 位的值
* _w_ 用于 _16_ 位的值
* _b_ 用于 _8_ 位的值

> 实际上, _x_ 的选择比这更多, 但是现在只涉及上面几个

<!--more-->


使用 _mov_ 指令时, _source_ 和 _destination_ 的组合必须满足一定的规则

* 立即数只能作为 _source_ , 此时 _destination_ 只能是
    * 通用寄存器
    * 内存地址
* 寄存器间可以任意组合
* 当 _source_ 或 _destination_ 中其中一个是内存地址时, 另一方只能是
    * 通用寄存器
    * 段寄存器
* 都是内存地址

