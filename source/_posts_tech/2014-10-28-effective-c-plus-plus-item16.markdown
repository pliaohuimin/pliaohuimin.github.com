---
layout: post
title: "effective_c++:item16 使用new和delete时应保持用法一致"
date: 2014-10-28 20:03
comments: true
categories: effective_c++
---
当你使用new时，会发生两件事情，一个是分配内存，另一个是调用一个或者多个的构造函数。当你使用delete时，会发现另外两件事情，一个或者多个析构函数被调用，然后是内存空间的释放。这个涉及到一个问题，该删除多少个对象，这问题会决定多少个析构函数被调用。

实际上，这问题很简单：删除时指定是删除单个对象或者是一个数组。这个问题很重要，因为单对象的内存布局和数组对象的内存布局是不一样的。

规则很简单：如果你在new表达式中使用了[]，必须在delete中也使用[]。如果你在new中没使用[]，那么在delete中也不要使用[]。

在使用typedef时，需要特别注意，因为它意味着typedef的作者必须明文说明使用哪个形式的delete。来看看下面的例子：
>
    typedef std::string AddressLines[4];
    std::string *pal = newAddressLines;

这时，必须使用数组形式的delete：
>
    delete pal; //undefined
    delete [] pal; //fine

避免这个问题也很简单，方法就是使用C++库的string和vector。

###Things to Remember
- 如果你在new表达式中使用了[]，必须在delete中也使用[]。如果你在new中没使用[]，那么在delete中也不要使用[]。
