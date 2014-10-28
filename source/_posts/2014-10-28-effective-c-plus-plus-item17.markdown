---
layout: post
title: "effective_c++:item17 在独立语句中将new出来的对象存放在智能指针中"
date: 2014-10-28 20:03
comments: true
categories: effective_c++
---
考虑一下代码：
>    
    int priority();
    void processWidget( std::tr1::shared_ptr<Widget> pw, int priority );
    processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());

在调用processWidget之前，编译器必须生成代码来完成下面三个事情：
- 调用priority
- 执行new Widget
- 调用 tr1::shared_ptr 构造函数

我们只能确保"new Widget"会在调用tr1::shared_ptr构造函数前被调用，但priority的调用会在第一，第二还是第三，我们是无法确定的。如果priority的调用是在第二，即：
- 执行new Widget
- 调用priority
- 调用 tr1::shared_ptr 构造函数

请考虑一下在调用priority时抛出异常，会出现什么情况。这种情况下，依旧会出现资源泄露。

想避免类似问题，方法也很简单：使用独立的语句来创建Widget并将它保存在智能指针中。然后再把智能指针传递给processWidget:
>    
    std::tr1::shared_ptr<Widget> pw( new Widget);
    processWidget(pw,priority());

###Things to Remember
- 在独立语句中，将new出来的对象保存在智能指针中。不这么做的话，一旦有异常抛出，可能会导致资源泄露。
