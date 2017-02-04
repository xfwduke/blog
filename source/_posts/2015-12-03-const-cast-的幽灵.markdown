title: const_cast 的幽灵
comments: false
date: 2015-12-03 22:58:58
tags:
  - C++ Primer
categories:
  - C/C++
---

{% blockquote Stanley B.Lippman; Josée Lajoie; Barbara E.Moo, C++ Primer 4.11.3 %}
_const_cast_ 用来去掉某个对象的 _const_ 性质
……
……
然而如果对象是一个常量，再使用 _const_cast_ 执行写操作就会产生未定义的结果。
{% endblockquote %}
    
    
<br>

这里提到的 `未定义的结果` 非常的诡异, 只看 _C++_ 代码的话, 世界观都被颠覆了

<!--more-->

## 幽灵

{% codeblock 幽灵代码 lang:cpp %}
#include <iostream>

using namespace std;

int main (int argc, char **argv)
{
  const int a {10};
  int *ptr {const_cast<int *>(&a)};
  *ptr = 20;
  cout << "&a: " << &a << ", a: " << a << endl;
  cout << "ptr: " << ptr << ", *ptr: " << *ptr << endl;
  return 0;
}
{% endcodeblock %}

这段代码是不对的——试图用 _const_cast_ 绕过 _const_ 限制去修改常量值——但代码是可以编译运行的, 只是结果非常奇怪

{% codeblock 诡异的结果 %}
&a: 0x7fff4fd0c6cc, a: 10
ptr: 0x7fff4fd0c6cc, *ptr: 20
{% endcodeblock %}

当看到这个结果的时候, 简直是条件反射的说出了 ___WTF___

同一个内存地址, 怎么可能有两个不同的值?

## 抓鬼

{% codeblock 重格式化代码 lang:cpp %}
#include <iostream>
using namespace std;
int main() {
    const int a {10};
    int *ptr {const_cast<int*>(&a)};
    *ptr = 20;
    printf("%x: %d\n", &a, a);
    printf("%x: %d\n", ptr, *ptr);
    return 0;}
{% endcodeblock %}

 用 _printf_ 代替 _cout_, 这样汇编代码更简单; 同时去掉了不必要的格式化, 这样代码的行号才能和调试信息对上

{% codeblock 带调试信息的汇编代码 %}
.loc 1 4 0
movl    $10, -20(%rbp)
.loc 1 5 0
leaq    -20(%rbp), %rax
movq    %rax, -16(%rbp)
.loc 1 6 0
movq    -16(%rbp), %rax
movl    $20, (%rax)
.loc 1 7 0
leaq    -20(%rbp), %rax
movl    $10, %edx
movq    %rax, %rsi
movl    $.LC0, %edi
movl    $0, %eax
call    printf
.loc 1 8 0
movq    -16(%rbp), %rax
movl    (%rax), %edx
movq    -16(%rbp), %rax
movq    %rax, %rsi
movl    $.LC0, %edi
movl    $0, %eax
call    printf
.loc 1 9 0
movl    $0, %eax
{% endcodeblock %}

直到汇编代码第 _8_ 行, 完成了
* 初始化 _a_
* 初始化 _*ptr_
* 通过 _ptr_ 修改 _a_ 的值

此时 
* _a_ 的值确实变成了 _20_ , 即 地址是{% raw %}$ \%rbp - 0x20 ${% endraw %} 的内存处的值
* 栈 {% raw %}$ \%rbp - 0x16 ${% endraw %}的值是 _&a_ 即 _ptr_ 的值

再看接下来的 _printf_ 部分
* 第 _10_ 和 _17_ 行 分别获得了 _&a_ 和 _ptr_ 的值, 这两个值是一样的
* 关键就是第 _11_ 行, 这里直接用到了立即数 _10_——<font color=red>__这就是造成同一内存地址有两个不同值的假象的幽灵__</font>

## 后记

鬼抓到了, 但是不理解为什么要这样处理