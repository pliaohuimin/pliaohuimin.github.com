---
layout: post
title: "effective_c++:item15 在资源管理类中提供原资源的存取接口"
date: 2014-10-28 20:02
comments: true
categories: effective_c++
---
很多时候，APIs要求原资源作为参数，这时，你需要一种能将RAII类转换成原资源的途径。一般有两种方法：显性转换和隐性转换。

tr1::shared_ptr和auto_ptr都提供了一个get的成员方法来实现转换，返回智能指针指向对象的祼指针：
> int days = daysHeld(pInv.get());

一般智能指针中都重载了->和*操作，这允许了隐性转换。

采用显性转换时（如get()），有时用户觉得使用起来不方便，就懒得使用资源管理类。使用隐性转换（如重载()操作符）会使API的调用看起来比较简单和自然。

但这会增加犯错的机会。
>    
    FontHandle getFont();
    Font f1(getFont());
    ...
    FontHandle f2 = f1; // meant to copy a Font object, but insterad implicitly converted f1 into its underlying FontHandle,then copy that

现在一个FontHandle被f1管理着，但同时FontHandle也被f2使用。这几乎从来都不是什么好兆头。例如，当f1销毁时，资源会被释放，而f2则悬挂着，指向非法空间。

使用显性还是隐性转换，取决于具体的使用场景。一般来说，显性转换更好些，因为它能尽可能地减少在你不想转换时被转换的机会。

RAII类返回原资源是对封装的破坏，但可能并不是设计上的灾难。RAII类不是用来封装某些东西的。它们的存在是为了确保一个特定的行为（如资源释放）的发生。像大部分设计得好的类，它隐藏了用户不需要看到的部分，但也提供给用户对需要存取的数据的接口。

###Things to Remember
- APIs经常要求原资源，所以每个RAII类都必须提供一个获取它管理的原资源的方法。
- 存取方法可以通过显性或者隐性的转换。一般来说，显性更安全些，但隐性对用户来说更加方便。
