---
layout: post
title: literal type
comments: false
date: 2016-02-04 21:35:26
tags:
  - The C++ Programming Language 4th
categories:
  - C/C++
---

_literal type_ 这个概念可以说是专为 _constexpr_ 而生

{% blockquote @LiteralType http://en.cppreference.com/w/cpp/concept/LiteralType %}
Literal types are the types of constexpr variables and they can be constructed, manipulated, and returned from constexpr functions.
{% endblockquote %}

<br>

<!--more-->

_literal type_ 的定义是（只涉及 _C++ 11_ 中有的）[^1]

[^1]:[http://en.cppreference.com/w/cpp/concept/LiteralType](http://en.cppreference.com/w/cpp/concept/LiteralType)

* _scalar type_
* _reference type_
* _an array of literal type_
* _class type that has all of the following properties_
  * _has a trivial destructor_
  * _is ether_
      * _an aggregate type_
      * _a type with at least one constexpr (possibly template) constructor that is not a copy or move constructor_
  * _all non-static data members and base classes are of non-volatile literal types_


## _literal type class_

用户自定义类要成为 _literal type_ 必须

* 只能有 _trivial destructor_ —— {% post_link trivial-type %}
* 非静态数据成员和基类必须是 _non-volatile literal type_

同时

* 要么是 _aggregate type_ —— {% post_link aggregate-initialization %}
* 要么至少有一个 _constexpr constructor_ —— 不能是复制和移动构造函数
