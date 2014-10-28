---
layout: post
title: "effective_c++:item14 认真考虑资源管理类的拷贝行为"
date: 2014-10-28 20:01
comments: true
categories:  effective_c++
---
有时，我们需要创建自己的资源管理类。
例如，你希望通过C API来操作锁对象的加锁和解锁功能。
> void lock(Mutex* pm);
> void unlock(Mutex* pm);

为了确保你不会忘记对已经加锁的对象解锁，你想创建一个类来管理锁。基本需求就是在该类的构造函数中加锁，在析构函数中解锁：
>    
    class Lock {
        public:
            explicit Lock(Mutex* pm): mutexPtr(pm)
            {
                lock(mutexPtr);
            }
            ~Lock() { unlock(mutexPtr); }
        private:
            Mutex* mutexPtr;
    };

用户使用RAII风格来使用Lock：
>    
    Mutex m;
    {
        Lock ml(&m);
        ...
    }

这运行得很好，但是当锁被拷贝时呢？
>    
    Lock ml1(&m);
    Lock ml2(ml1);

这是类的编写者需要考虑的问题：当一个RAII对象被拷贝时，应该如何处理？大部分情况下，你会选用以下几个方式的一个来处理：
- 禁止拷贝。如果RAII对象拷贝没有意义时，你应该这么做。
- 记录资源引用数，当最后一个引用对象析构时才释放资源。

tr1::shared_ptr允许指定一个"deleter"--当引用数量为0时，调用指定函数或者函数对象。
>    
    class Lock{
    public:
        explict Lock(Mutex* pm):mutexPtr(pm,unlock)
        {
            lock(mutexPtr.get());
        }
    private:
        std::tr1::shared_ptr<Mutex> mutexPtr;
    };

- 对资源进行深拷贝。当你想拥有多个资源的拷贝，采用资源管理类只是为了确保每个资源都会被释放时，可以对管理的资源对象进行深拷贝。

-  移交控制权。如*auto_ptr*，拷贝方法（构造函数或者拷贝赋值操作）可能由编译器生成，如果这不是你想要的版本，你需要重写它们。

###Things toRemember###
- 资源的拷贝行为决定了RAII对象的拷贝行为
- 常见的RAII类拷贝行为为不允许拷贝和使用引用计数的方式，但也有一些其它的选择。


        
