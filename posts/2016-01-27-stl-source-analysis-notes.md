---
title: 『STL源码剖析』笔记
date: 2016-01-27 15:30:04
tags:
  - stl
  - 内存管理
  - 迭代器
  - c++
categories:
  - 程序设计
---

## STL

本书的开头，千言万语汇成一句话：STL很牛逼。STL有多种实现，本书剖析的SGI版本可读性很高，也是被GCC使用的版本。

这本书我粗略翻了一遍，个人认为本书的第二章和第三章是精华，也是我读得最仔细的部分。后面章节的内容则是数据结构和算法的具体实现。本书非常适合与讲算法的书搭配学习，看看工业级别的代码是怎么写的。

<!--more-->

## allocator

内存管理是非常重要的工作，C++内存的配置和释放是通过new和delete完成的。new语句包含两阶段操作：（1）调用`::operator new`配置内存；（2）调用类的构造函数构造对象内容。delete语句也包含两阶段操作：（1）调用类的析构函数将对象析构；（2）调用`::operator delete`释放内存。

为了精密分工，STL allocator将这两阶段操作区分开。

+ 内存配置：`alloc::allocate()`
+ 内存释放：`alloc::deallocate()`
+ 对象构造：`::construct()`
+ 对象析构：`::destroy()`

在具体实现上，上述的`construct()`接受一个指针和一个初值，该函数的用途是将初值设定到指针所指的空间上。这通过C++的placement new运算子实现。

`destroy()`通过`value_type()`获取迭代器所指对象的型别，再利用`__type_traits<T>`判断该类型的析构函数是否无关痛痒（所谓trivial destructor）。若是，则什么也不做；若否，则遍历整个范围，析构每个对象。因为一次次地调用无关痛痒的析构函数，会损害效率。

考虑到内存碎片问题，SGI设计了双层级配置器，第一级配置器直接使用`malloc()`和`free()`，第二级配置器视情况采用不同的策略：当配置区块足够大，调用第一级配置器；当配置区块较小时，采用负责的内存池（memory pool）方式。内存池的代码很值得学习。

除了`construct()`和`destroy()`之外，STL还定义了另外三个作用于未初始化空间上的函数，将内存的配置和对象的构造行为分开：

+ `uninitialized_copy()`
+ `uninitialized_fill()`
+ `uninitialized_fill_n()`

## iterator

迭代器设计思维是STL的关键所在。STL的中心思想在于：将数据容器（containers）和算法（algorithms）分开，彼此独立设计，最后再以一帖胶合剂将它们撮合在一起。迭代器就是这帖胶合剂。

迭代器的行为类似于指针，指针的各种行为中，最重要的即为内容提领（dereference）和成员访问（member access），迭代器最重要的编程工作就是对`operator*`和`operator->`进行重载。每种STL容器都提供专属迭代器。设计迭代器是容器的责任，只有容器本身才知道该设计出怎样的迭代器来遍历自己，并执行迭代器该有的各种行为（前进、后退、取值、取用成员……）。至于算法，完全可以独立于容器和迭代器之外自行发展，只要设计时以迭代器为对外接口就行。

迭代器所指对象的型别，成为迭代器的value type。型别的获取是通过C++的模板偏例化（template partial specialization）特性实现的，该方法称之为**traits编程技法**。例如，原生指针`int *`虽然不是一种class type，但可以通过traits获取其value type，代码如下：

```cpp
template <class T>
struct iterator_traits<T *> {
    typedef T value_type;
};
```

最常用的相应型别有五种：value type, difference type, pointer, reference, iterator catagory。

```cpp
template <class I>
struct iterator_traits {
    typedef typename I::iterator_category iterator_category;
    typedef typename I::value_type        value_type;
    typedef typename I::difference_type   difference_type;
    typedef typename I::pointer           pointer;
    typedef typename I::reference         reference;
};
```

## 序列式容器

### vector

vector的数据安排以及操作方式和array很类似，唯一差别在于空间运用的灵活性。array是静态空间，一旦配置了就不能改变。vector是动态空间，随着元素的加入，它的内部机制会自行扩充空间以容纳新元素。

当以`push_back()`将新元素插入vector的尾端时，如果没有备用空间，就扩充空间（重新配置，移动数据，释放原空间）。动态增加大小，不是在原空间之后接续新空间，而是以原大小的两倍另外配置一块较大空间，然后将原内容拷贝过来，然后才开始在原内容之后构造新元素，并释放原空间。所以，一旦引起空间重新配置，指向原来vector的所有迭代器就失效了。

### list

STL的list是用双向链表实现的，而且是一个环状双向链表。STL有个用单向链表实现的slist。

### deque

vector是单向开口的连续线性空间，deque则是一种双向开口的连续线性空间。即deque可以在头尾两端分别做元素的插入和删除操作。deque和vector的最大差异，一在于deque运行常数时间内对起头端进行元素的插入或移除操作，二在于deque没有所谓容量的观念，因为它是动态地以分段连续空间组合而成，随时可以增加一段新的空间并链接起来。

deque由一段一段的定量连续空间构成。一旦有必要在deque的前端或尾端增加新空间，便配置一段定量连续空间，串接在整个deque的头段或尾端。deque在这些分段的定量连续空间上维护其整体连续的假象，并提供随机存取的接口。避开了vector的“重新配置、复制、释放”的轮回，代价则是复杂的迭代器架构和更大的代码量。

由于deque双向开口的特性，SGI STL在缺省情况下以deque作为stack和queue的底部结构。

## 关联式容器

STL提供8个关联容器。它们的不同体现在三个维度上：

+ 或是一个set，或是一个map
+ 或要求不重复的关键字，或允许重复关键字。允许重复关键字的容器名称中包含multi
+ 按顺序保存元素，或无序保存。无序保存关键字的容器名称中包含unordered

无序保存关键字的容器底层使用哈希表（hash table）实现；按序保存关键字的容器底层使用红黑树（Red-Black Tree）实现。

## 算法

待续。

## 仿函数functor

待续。

## 配接器adapter

待续。
