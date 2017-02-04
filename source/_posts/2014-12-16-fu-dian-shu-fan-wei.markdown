---
layout: post
title: "浮点数范围"
date: 2014-12-16 20:49:26 +0800
comments: false
categories: 
  - 汇编
tags: 
  - Professional Assembly Language
---

浮点数的实现比整数复杂很多, 并且不同类型的浮点数(单精度或双精度)的挑选, 也不仅仅和最大值有关

在书上看到和浮点相关的内容, 对于单精度浮点数绝对值最大, 最小值范围有疑惑, 所以查阅了一些文档, 

按数值的最小粒度, 可以把浮点数分为两类[^1]
* 规约浮点数 ( _normalized number_ )
* 非规约浮点数 ( _denormalized number_ )

<!--more-->

### 规约单精度浮点数 ( _normalized number_ )

[^1]:[IEEE 754-1985](http://en.wikipedia.org/wiki/IEEE_754-1985#Single-precision_32-bit)  

单精度浮点数( _float_ ) 用 _32 bit_ 表示, 具体的布局是 [^2]

[^2]:[IEEE 754](http://zh.wikipedia.org/wiki/IEEE_754)

<table border = '1' align="center">
<tr>
<th style="text-align:center">31</th>
<th style="text-align:center">30</th>
<th style="text-align:center">29</th>
<th style="text-align:center">~</th>
<th style="text-align:center">24</th>
<th style="text-align:center">23</th>
<th style="text-align:center">22</th>
<th style="text-align:center">21</th>
<th style="text-align:center">~</th>
<th style="text-align:center">01</th>
<th style="text-align:center">00</th>
</tr>
<tr>
<td style="text-align:center">符号位 (sign bit)</td>
<td style="text-align:center" colspan='5'>指数位 (Exponent bit)</td>
<td style="text-align:center" colspan='5'>尾数位 (Significand bit)</td>
</tr>
</table>

* 符号位取 _0_ 时为正值
* 指数位有 _8 bits_
    * _0_ 和 _255_ 为特殊值, 不能使用
    * 用于生成浮点数的指数值需要做 _-127_ 偏移, 所以实际指数值范围是 _[ -126, 127 ]_
* 尾数位有 _23 bits_
    * 对于规约浮点数, 全用于表示小数点后的有效值, 而小数点前有效值固定为 _1_
    * 对于非规约浮点数, 先不做讨论 
    

#### 指数的计算 _Exponent_

指数的计算非常简单, 将二进制数转换为十进制数, 并注意定义中的偏移即可

#### 尾数计算 _Significand_

规约浮点数的尾数位全部是小数点后的有效数字, 且第 _22_ 位是小数点后第 _1_ 位, 以此类推. 对于上面表格中的尾数位, 从左向右重新编号如下

<table border = '1' style="text-align:center">
<tr>
<th style="text-align:center">0</th>
<th style="text-align:center">1</th>
<th style="text-align:center">~</th>
<th style="text-align:center">21</th>
<th style="text-align:center">22</th>
<tr>
<td style="text-align:center" colspan='5'>尾数位</td>
</tr>
</tr>
</table>

这里的尾数是用二进制表示的小数, 同样需要转换为十进制, 再考虑到规约浮点固定有 _1_ 做为尾数, 所以
	
* 取 _i_ 为上表中的编号
* 当某位为 _1_ 时, 对应的 _bit = 1_; 当某位为 _0_ 时, 对应的 _bit = 0_

最终有如下尾数计算公式

{% raw %}
$$
Significand = 1 + \sum_{i=0}^{22} {bit}_i \times 2^{-(i+1)}
$$
{% endraw %}

#### 规约浮点数值计算

把上面各部分合起来, 有如下浮点数值计算公式 —— 需要重点注意的是, 指数对应的底 ( _Base_ ) 为 _2_

{% raw %}
$$
float = (-1) ^ {sign} \times {Significand} \times 2 ^ {Exponent}
$$
{% endraw %}

#### 数值范围

先忽略符号位, 只计算绝对值的上下限
	
* 下限

{% raw %}
$$
{Significand}_{min} = 1
$$

$$
{Exponent}_{min} = {-126}
$$

$$
{float}_{min} = 1 \times 2^{-126} \approx {1.17549} \times {10}^{-38}
$$

{% endraw %}

* 上限

{% raw %}

$$
{Significand}_{max} = 1 + \sum_{i=0}^{22} {1} \times 2^{-(i+1)} \approx {1.99999976158142} \approx 2
$$
	
$$
{Exponent}_{max} = 127 
$$

$$
{float}_{max} = 2 \times 2 ^ {127} = 2 ^ {128} \approx {3.40282} \times {10} ^ {38}
$$

{% endraw %}
	
* 绝对值范围

{% raw %}

$$
{float} \in \lbrack 2 ^ {-126} , 2 ^ {128} \rbrack
$$

{% endraw %}
	
或者  
{% raw %}
$$
{float} \in \lbrack {1.17549} \times {10}^{-38} , {3.40282} \times {10} ^ {38} \rbrack
$$
{% endraw %}
  
对于规约单精度浮点, 有这样几个特点 (均只讨论绝对值)

* 最小值是 $ 2 ^ {-126} $
* 精度 —— 相邻的两个数的差值是 $ 2 ^ {-149} $  
  最小值和精度不相等, 且相差倍数极大 —— 这两个特性的另外一个含义是规约单精度浮点在小于其最小值后, 以极快的速度溢出到 _0_ , 这个速度比起大于最小值时的收敛速度有些过于的快了[^2]



对比整数类型, 最小值和粒度都是 _1_

### 非规约单精度浮点 ( _denormalized number_ )




正是由于浮点有上面的特性, 才出现了非规约的概念 —— 为了补充 _0_ 到绝对值最小值之间的空白

实际的非规约定义比较复杂, 这里可以做些简化 [^2]



* 指数位全为 _0_ , 且此时指数值记为 _-126_
* 尾数位含义不变
* 尾数小数点前的值可以为 _0_

按照这样的定义, 用类似方法计算绝对值范围是

{% raw %}
$$
{float} \in \lbrack 2 ^ {-149},2 ^ {-126} \rbrack
$$
{% endraw %}
	
或者

$$
{float} \in \lbrack {1.4013} \times 10 ^ {-45}, {1.17549} \times {10}^{-38} \rbrack
$$这样补充之后, 单精度浮点的最小值和精度大小一样了, 并且收敛速度变得平滑
	
### 编程中的使用


{% blockquote Tim Wilson, Do not use denormalized numbers https://www.securecoding.cert.org/confluence/display/seccode/FLP05-C.+Don't+use+denormalized+numbers %}
Do not produce code that could use denormalized numbers. If floats are producing denormalized numbers, use doubles instead
{% endblockquote %}


因为规约双精度浮点的精度比非规约单精度浮点的精度高很多[^3]

[^3]:[双精度浮点数](http://zh.wikipedia.org/wiki/雙精度浮點數)

### 双精度浮点数

基本概念差不多, 只是变成了 _64 bits_

### 参考文档
[IEEE floating point](http://en.wikipedia.org/wiki/IEEE_floating_point)   
[Denormal number](http://en.wikipedia.org/wiki/Denormal_number)    
[Why does changing 0.1f to 0 slow down performance by 10x?](http://stackoverflow.com/questions/9314534/why-does-changing-0-1f-to-0-slow-down-performance-by-10x/9314926#9314926)   





