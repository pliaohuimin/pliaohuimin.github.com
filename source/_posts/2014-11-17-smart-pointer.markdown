---
layout: post
title: "各智能指针对比"
date: 2014-11-17 16:56
comments: true
categories: IT
---
### std::auto_ptr
- 复制行为：原来指针变成null,而复制所得的指针将取得资源的唯一拥有权
```c++
#include <stdio.h>
#include <memory>                                                                                                               

using namespace std;

int main( int argc, char* argv[] )
{
    auto_ptr<int> a (new int);
    *a = 10;
    printf( "a's value:%d\n", *a );
    printf( "a's addr:%p\n", &(*a) );

    auto_ptr<int> b = a;
    printf( "b's addr:%d\n", *b );
    printf( "a's addr:%p\n", &(*a) );

    return 0;
}

```

### boost::shared_ptr
- 复制行为：添加引用数
- 删除时机：离开作用域时，引用数减一。没人指向它时（即引用数为0时），删除对象
```c++
#include <stdio.h>
#include <boost/shared_ptr.hpp>

using namespace std;

int main( int argc, char* argv[] )
{
    boost::shared_ptr<int> a (new int);
    *a = 10;
    printf( "a's value:%d\n", *a );
    printf( "a's addr:%p\n", &(*a) );

    printf( "a's use_count:%d\n", a.use_count() );
    boost::shared_ptr<int> b = a;
    printf( "b's addr:%d\n", *b );
    printf( "a's addr:%p\n", &(*a) );
    printf( "a's use_count:%d\n", a.use_count() );
    printf( "b's use_count:%d\n", b.use_count() );                                                                              

    return 0;
}

```

### boost::scoped_ptr
- 复制行为：不能拷贝

### boost::weak_ptr
- 用途：
    1. weak_ptr被设计为与shared_ptr共同工作，可以从一个shared_ptr或者另一个weak_ptr对象构造，获得资源的观测权。防止循环引用。
    2. 防止使用boost::shared_ptr时，使对象的生命期被意外延长。
- 复制行为：weak_ptr没有共享资源，它的构造不会引起指针引用计数的增加。
- weak_ptr是为了配合shared_ptr而引入的一种智能指针，它更像是shared_ptr的一个助手而不是智能指针，因为它不具有普通指针的行为，没有重载operator*和->,它的最大作用在于协助shared_ptr工作，像旁观者那样观测资源的使用情况
