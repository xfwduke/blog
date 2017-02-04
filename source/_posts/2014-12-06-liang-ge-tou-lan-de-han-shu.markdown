---
layout: post
title: "两个偷懒的函数"
date: 2014-12-06 22:04:55 +0800
comments: true
categories: 
  - 汇编
tags: 
  - Professional Assembly Language
---

跟着 _Professional Assembly Language_ 开始看, 写代码的时候经常要用到 _as_ 然后 _ld_ , 还挺麻烦的, 而且一不小心把源代码文件接在了 _-o_ 后面真是想吐血

所以写了两个函数帮忙搞定, 然后再用第三个函数串起来, 扔到了 _.bashrc_ 里面

<!--more-->

{% codeblock 添加到 bashrc lang:bash %}
__gas32 () {
    filename=$1
    ext_name=${filename##*.}
    if [ $ext_name == 's' -o $ext_name == 'S' ]
    then
        base_name=${filename%.$ext_name}
        as --32 -gstabs+ $filename -o $base_name.o
        if [ $? -eq 0 ]
        then
            objname=$base_name.o
        fi
    else
        echo "$filename may be not an assemble source"
    fi
}
__ld32 () {
    filename=$1
    ext_name=${filename##*.}
    if [ $ext_name == 'o' ]
    then
        base_name=${filename%.$ext_name}
        ld -m elf_i386 $filename -o $base_name
    else
        echo "$filename may be not an object"
    fi
}
asbuild () {
    unset objname
    __gas32 "$1"
    if [ ! -z  $objname ]
    then
        __ld32 "$objname"
    fi
}
{% endcodeblock %}

* 由于机器是 _64bit_ 系统, 书上主要讲 _32bit_ 变成, 为了避免因为字长导致的错误, _as_ 使用了 _--32_ 参数
* _as_ 使用了 _-gstabs+_ 来添加调试信息, 方便 _gdb_
* _ld_ 使用了 _-m elf___i386_ 来链接 _32bit_ 程序
* 最终在命令行用 `asbuild source.s` 就好了


实际上, _gcc_ 可以直接搞定上面的事情

```
gcc -o excutable source.s
```

有几点要注意的

* 必须把程序的`开始位置标签`设置为 _main_   详见:{% post_link kai-shi-wei-zhi-biao-qian 开始位置标签 %} 
* 因为是在 _64 bit_ 机器上编译 _32 bit_ 代码, 必须安装对应版本的库文件, _debian_ 上是 _libc6-dev-i386 libc6-i386_
* _gcc_ 必须使用 _-m32_ 参数