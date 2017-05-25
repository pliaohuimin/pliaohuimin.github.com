---
layout: post
title: "effective_c++:item13 使用对象来管理资源"
date: 2014-10-28 19:35
comments: true
categories: effective_c++
---

##问题
有一个基类：
> class Investment {...};

通过工厂函数来提供Investment实例：
> Investment* createInvestment();

这样一来，createInvestment的调用者在使用完该函数提供的对象指针后，需要负责把它delete掉。

考虑以下的使用场景：
>    
    void f()
    {
        Investment* pInv = createInvestment();
        ....  // some code
        delete pInv;
    }

以下场景会导致删除pInv失败：
- f函数在...中返回
- f函数在...中抛出异常

当然，细心地编程能够避免以上错误的发生。但代码在经过几次的修改后呢？在软件的维护过程中，有人可能会在中间添加新的return或者continue而没有很好地处理资源问题。更糟糕的是，在...部分以前可能没有使用异常机制，突然有人觉得应该加入以提高性能。简单地依赖于人员记得去删除资源是不可行的。


###解决方案一
为了确定获得createInvestment返回的资源总能被释放，我们需要把资源放进一个对象中，该对象的析构函数总能自动地释放该资源。

很多用堆动态分配出来的资源，只用于一个特定的代码块中，当控制流离开代码块或者函数时，需要被释放。标准库的*auto_ptr*就是为这种场景量身定制的。*auto_ptr*是一个类指针对象，它的析构函数会自动删除它指向的资源。下面是如何使用*auto_ptr*来避免f函数潜在的内存泄露问题：

>        
    void f()
    {
        std::auto_ptr<Investment> pInv(createInvestment());
    ...
    }

这个简单的例子展示了用对象来管理资源中两个关键概念：

资源总是在获取时立即放进资源管理对象中。上面例子中，资源中由*createInvestment()返回时，被用来初始化管理它的*auto_ptr*对象。这种方法叫做资源获得即初始化（RAII），因为在获得一个资源的同时，又初始化了资源管理的对象。有时资源是通过赋值传给资源管理对象。

由于*auto_ptr*对象析构时总会删除它指向的对象，很重要的一点就是需要保证不能有多于1个的*auto_ptr*指向同一个对象。不然，对象就会被删除超过一次。为了防止类似问题，*auto_ptr*有一个特性：对资源管理对象进行拷贝操作时，会把原来的管理对象设置为null的，也就是说，资源管理对象从原来对象转移到新的资源管理对象。

>    
    std::autoptr<Investment> pInv1(createInvestment());  //pInv1 points to the objct return from createInvestment
    std::autoptr<Investment> pInv2(pInv1); //pInv2 now points to the object;pInv1 is now null
    pInv2 = pInv2; //now pInv1 points to the object,and pInv2 is null

这种拷贝特性意味着*auto_ptr*没法很好地管理所有的动态分配资源。例如STL容器要求它们的内容必须有正常的拷贝行为表现。

###解决方案二
另外一个替换方案就是reference-counting smart pointer(RCSP)。然而*RSCP*无法处理循环引用 的问题（例如，两个没用的对象互相指向对方）。

TR1的tr1::shared_ptr就是一个RCSP。所以你可以这样写：
>    
    void f()
    {
        std::tr1::shared_ptr<Investment> pInv(createInvestment());
        ...
    }

代码看起来和使用*auto_ptr*一样，但拷贝*shared_ptr*的行为看起来自然一些：
>    
    void f()
    {
        std::tr1::shared_ptr<Investment> pInv1(createInvestment()); //pInv1 points to the objct return from createInvestment
        std::tr1::shared_ptr<Investment> pInv2(pInv1);  //both pInv1 and pInv2 now point to the object
        pInv1 = pInv2;  // nothing changed
        ...
    }

由于*shraed_ptr*工作起来与STD容器所期望的一致，它能在STL容器中使用。

*auto_ptr*和*tr1::shared_ptr*都在析构函数中使用了*delete*而不是*delete []*，这意味着无法使用它们来释放数组资源。

如果你想处理数组对象，你可以使用*boost::scoped_array*和*boost:shared_array*。

###Thigns to Remember###
- 以防止资源泄露，在RAII对象构造函数中来获取资源，在它的析构函数中来释放资源。
- 两个有用的RAII类是TR1::shared_ptr和auto_ptr，通常来说，需要拷贝时，TR1::shared_ptr是更好的选择。拷贝auto_ptr会把原来的设置为null。
