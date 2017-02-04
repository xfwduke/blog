---
layout: post
title: constexpr
comments: false
date: 2016-02-10 14:06:14
tags:
  - The C++ Programming Language 4th
categories:
  - C/C++
---

_C++ 11_ 中新增的 _constexpr_ 关键字用于声明可以在编译期求值的变量或函数——已有的 _const_ 关键字是承诺不再修改变量的值—— _constexpr_ 变量也不能修改

_constexpr_ 按场景可以细分为

* _constexpr variable_
* _constexpr function_
* _constexpr constructor_

详细的规则比较复杂，可以看下 [constexpr specifier](http://en.cppreference.com/w/cpp/language/constexpr)[^1]

[^1]:[http://en.cppreference.com/w/cpp/language/constexpr](http://en.cppreference.com/w/cpp/language/constexpr)

<!--more-->

## _constexpr variable_

_constexpr variable_ 要求感觉比较抽象，不太容易和实际联系起来


> ___its type must be a LiteralType___


这个比较好理解，不是所有类型都可以用于 _constexpr_ —— _std::string_ 就不行，因为它是 _non-literal_


> ___it must be immediately constructed or assigned a value___


这一要求和 _const_ 一样——不能只声明而不初始化


> ___the constructor parameters or the value to be assigned must contain only literal values, constexpr variables and functions___

这一条字面意思是 构造一个 _constexpr variable_ 时，构造函数的参数只能包含 _literal values、constexpr variables、constexpr functions_ 

但实际上还表示 不能用非 _constexpr variables_ 来初始化 _constexpr variables_

{% codeblock lang:cpp 错误 %}
int a = 10;
constexpr int b = a;
{% endcodeblock %}

上面的代码就违反了这一要求，换一种写法更明显


{% codeblock lang:cpp 换一种写法 %}
int a = 10;
constexpr int b(a);
{% endcodeblock %}


> ___the constructor used to construct the object (either implicit or explicit) must satisfy the requirements of constexpr constructor. In the case of explicit constructor, it must have constexpr specified___

这个简单了，自定义类必须有 _constexpr constructor_ 并被调用

## _constexpr function_

___constexpr function___ __默认的就是内联函数__

_constexpr function_ 对函数体中的语句有严格的要求 （ _C++ 11_ ），可以是下列情况

* 空语句
* _static_assert_
* _typedef_ 和 _alias_ 
* _using_ 语句
* 有且仅有一个 _return_

在 _C++ 14_ 中则增加了

* 内联汇编
* _goto_
* _try block_
* 一些变量定义

如 _if、for、while_ 这样的复杂逻辑是不允许出现的


{% codeblock lang:cpp 计算斐波那契数列 %}
#include <iostream>

using namespace std;

constexpr unsigned long long fun(const unsigned long long a)
{
  return (a > 1 ? fun(a - 1) + fun(a - 2) : 1);
}

int main()
{
  constexpr unsigned long long r = fun(20);
  cout<<r<<endl;
  return 0;
}
{% endcodeblock %}

计算斐波那契数列算是能用 _constexpr function_ 实现的比较复杂的例子了——递归调用刚好能用一条语句搞定

{% codeblock 汇编代码 %}
_main:                                  ## @main
        .cfi_startproc
## BB#0:
        pushq   %rbp
Ltmp0:
        .cfi_def_cfa_offset 16
Ltmp1:
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
Ltmp2:
        .cfi_def_cfa_register %rbp
        subq    $48, %rsp
        movq    __ZNSt3__14coutE@GOTPCREL(%rip), %rdi
        movl    $10946, %eax            ## imm = 0x2AC2
        movl    %eax, %esi
        movl    $0, -20(%rbp)
        movq    $10946, -32(%rbp)       ## imm = 0x2AC2
        callq   __ZNSt3__113basic_ostreamIcNS_11char_traitsIcEEElsEy
        ...
        ...
        .cfi_endproc
{% endcodeblock %}

从汇编代码可以看到，编译后直接计算出了 _fun(20)_ 的结果，没有出现 _call_ 指令调用 _fun()_ 函数

{% codeblock 非 constexpr 版本 %}
_main:                                  ## @main
        .cfi_startproc
## BB#0:
        pushq   %rbp
Ltmp3:
        .cfi_def_cfa_offset 16
Ltmp4:
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
Ltmp5:
        .cfi_def_cfa_register %rbp
        subq    $48, %rsp
        movl    $20, %eax
        movl    %eax, %edi
        movl    $0, -20(%rbp)
        callq   __Z3funy
        movq    __ZNSt3__14coutE@GOTPCREL(%rip), %rdi
        movq    %rax, -32(%rbp)
        movq    -32(%rbp), %rsi
        callq   __ZNSt3__113basic_ostreamIcNS_11char_traitsIcEEElsEy
        ...
        ...
        .cfi_endproc
{% endcodeblock %}

再看非 _constexpr function_ 版本的汇编代码，第 _16_ 行调用了 _fun()_ 函数，在运行时计算结果

## _constexpr constructor_

构造函数也是函数，所以 _constexpr constructor_ 天然带有 _constexpr function_ 的各种限制，然后才是特有的限制

* 不能有虚基类
* 构造函数不能有 _function-try-block_ ——多用于处理构造函数异常的函数定义形式[^2]
* ...

更详细的定义参考[^1]


[^2]:[http://en.cppreference.com/w/cpp/language/function-try-block](http://en.cppreference.com/w/cpp/language/function-try-block)