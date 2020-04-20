---
title: C++右值引用简介
date: 2016-11-20 19:02:32
tags:
  - c++
  - 拷贝控制
  - 右值引用
  - 移动语义
  - 引用
categories:
  - 编程语言
---

本文译自：[A Brief Introduction to Rvalue Reference](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html)，有少量修改。

## 介绍

本文简要介绍了C++的新特性——右值引用（rvalue reference），更多细节请参考[这里](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html#Wrap_Up)。

## 右值引用

右值引用是一种复合类型，和传统的C++引用类似。为了更好地区分它们，我们将传统的引用称之为左值引用。引用一词同时包括左值引用和右值引用。

<!--more-->

在类型名前加上`&`符号表示左值引用：

```cpp
A a;
A &ref1 = a;   // ref1是左值引用
```

在类型前加上`&&`符号表示右值引用：

```cpp
A a;
A &&ref2 = a;  // ref2是右值引用
```

右值引用的行为和左值引用类似，但右值引用能够绑定一个临时量（右值），而（非常量的）左值引用不能绑定一个右值：

```cpp
A &ref3 = A();   // 错误：不能绑定一个右值
A &&ref4 = A();  // 正确：可以绑定一个右值
```

将左值引用和右值引用结合起来能够实现移动语义（move semantics）。使用右值引用还可以实现完美转发（perfect forwarding），完美转发是C++之前没能解决的问题。从普通程序员的角度看，通过右值引用可以实现更通用、执行效率更高的库。

## 移动语义

### 避免不必要的拷贝

拷贝控制和资源管理在C++中很重要。拷贝的花销可能比较大，比如，对`std::vector`类型来说，`v2 = v1`通常包含一次函数调用、一次内存分配和一个循环。如果我们真正需要两份`vector`拷贝，花销是可以接受的。但大多数情况下，我们并不需要两份拷贝：我们通常只需要将一个`vector`从一个地方拷贝到另一个地方，然后直接覆盖旧的那份拷贝。例如：

```cpp
template <typename T> 
swap(T &a, T &b)
{
    T tmp(a);  // 现在有两份a的拷贝
    a = b;     // 现在有两份b的拷贝
    b = tmp;   // 现在有两份tmp的拷贝
}
```

但是我们不需要任何`a`或`b`的拷贝，我们只想交换（swap）它们：

```cpp
template <typename T>
swap(T &a, T &b)
{
    T tmp(std::move(a));
    a = std::move(b);
    b = std::move(tmp);
}
```

上面的`move()`将它参数的值赋给目标，但不负责保留它参数的内容。所以，对一个`vector`参数，期望的结果是，`move()`将参数变成容量为0的`vector`以避免拷贝所有的元素。换句话说，`move`读完对象的数据后即销毁对象。

在这种情况下，我们可以优化`swap`操作，但我们不能对每个拷贝大对象的函数都进行这种操作。

右值引用的第一个作用是高效、简洁地实现`move()`。

### move

`move`函数很简洁，接受一个左值或右值参数，然后返回一个右值且不触发拷贝构造函数：

```cpp
template <typename T>
typename remove_reference<T>::type &&move(T &&a)
{
    return static_cast<typename remove_reference<T>::type &&>(a);
}
```

然后用户根据参数是左值或右值（比如拷贝构造符和赋值构造符）重载函数。当参数是左值，从参数中拷出数据。当参数是右值，从参数移出数据。

### 左值/右值的重载

考虑一个简单的句柄（handle）类，该类拥有一个资源并提供拷贝语义操作（拷贝构造函数和赋值构造函数）。例如，一个`clone_ptr`拥有一个指针，调用`clone()`执行拷贝：

```cpp
template <typename T>
class clone_ptr {
private:
    T *ptr;
public:
    // 构造函数
    explicit clone_ptr(T *p = 0) : ptr(p) {}

    // 析构函数
    ~clone_ptr() { delete ptr; }

    // 拷贝语义
    clone_ptr(const clone_ptr &p) : ptr(p.ptr ? p.ptr->clone() : 0) {}

    clone_ptr &operator=(const clone_ptr &p) {
        if (this != &p) {
            delete ptr;
            ptr = p.ptr ? p.ptr->clone() : 0;
        }
        return *this;
    }

    // 移动语义
    clone_ptr(clone_ptr &&p) : ptr(p.ptr) { p.ptr = 0; }

    clone_ptr &operator=(clone_ptr &&p) {
        std::swap(ptr, p.ptr);
        return *this;
    }

    // 其它操作
    T& operator*() const { return *ptr; }
    // ...
};
```

除了移动语义（move semantics）之外，其它的代码在C++书中都很常见（译者注：move semantics即为移动构造（赋值）函数，已加入C++11标准，详见[C++ Primer第5版](https://book.douban.com/subject/25708312/)）。`clone_ptr`的用户代码如下：

```cpp
clone_ptr<base> p1(new derived);  
// ...
clone_ptr<base> p2 = p1;          // p2和p1都拥有自己的指针 
```

需要注意，拷贝构造和赋值构造一个`clone_ptr`相对来说开销大。但如果已知被拷贝的对象是右值，可以通过“偷窃”右值的指针来避免开销大的`clone()`操作。上面的移动构造函数就是这么实现的，最后让左值处于默认构造状态。移动赋值运算函数的实现中仅仅和右值交换了状态。

于是，当拷贝一个右值的`clone_ptr`，或代码中能显式表明拷贝源是一个右值（使用`std::move`），操作执行得会更快一些：

```cpp
clone_ptr<base> p1(new derived);
// ...
clone_ptr<base> p2 = std::move(p1);  // 现在p2拥有p1之前拥有的指针 
```

对于由其它类构成的类（通过包含或继承），移动构造函数和移动赋值函数可以通过`std::move`函数实现：

```cpp
class Derived : public Base {
    std::vector<int> vec;
    std::string name;
    // ...
public:
    // ...
    // move semantics
    Derived(Derived &&x)               // rvalues bind here
        : Base(std::move(x)), 
          vec(std::move(x.vec)),
          name(std::move(x.name)) { }

    Derived &operator=(Derived &&x) {  // rvalues bind here
        Base::operator=(std::move(x));
        vec = std::move(x.vec);
        name = std::move(x.name);
        return *this;
    }
    // ...
};
```

匹配到其构造函数和赋值函数时，子类对象就会被当做右值。`std::vector`和`std::string`的代码中添加了移动操作，大大减少了拷贝操作。

注意上面的代码，在`move`函数内部，参数`x`被当作左值，虽然它的声明是右值参数。这就是为什么说是`move(x)`而不是`x`传递给了基类。这是移动语义的一个关键的安全特性，可以防止意外地将某些变量移动两次。所有的移动操作只作用在右值上，或者用`std::move`显示地转化为右值。如果有变量名，它就是个左值。

如果类型不包含资源（比如`std::complex`），这种情况不用管，从右值拷贝时，拷贝函数就已经是最优了。

### 可移动但禁止拷贝的类型

有些类型不适用拷贝语义，但也可以让其移动。比如：

* fstream
* unique_ptr（不可共享，不可拷贝）
* 表示进程运行的类型

将这些类型变成可移动的，它们的功能会大大增强。可移动但禁止拷贝的类型可以通过工厂函数（factory functons）返回：

```cpp
ifstream find_and_open_data_file(/* ... */)
// ...
ifstream data_file = find_and_open_data_file(/* ... */)  // 没有拷贝
```

上面的例子中，只要`ifstream`是右值，底层的文件句柄在对象之间传递。自始至终只存在一个底层的文件句柄，而且同一时间只有一个`ifstream`拥有它。

可移动但禁止拷贝的类型也可以安全地放在标准容器中。如果容器需要在内部“拷贝”元素（比如`vector`的reallocation），它就会移动而不是拷贝元素：

```cpp
vector<unique_ptr<base>> v1, v2;
v1.push_back(unique_ptr<base>(new derived()));  // OK，移动而不是拷贝
// ...
v2 = v1;             // 编译时错误，是不可拷贝的类型
v2 = move(v1);       // 移动OK，指针的拥有权转移给v2
```

很多标准算法从中受益，这不仅仅提高了执行效率（比如上面的`std::swap`实现），也使得这些算法可以用在可移动但禁止拷贝的类型上。例如，下面的代码对一个`vector<unique_ptr<T>>`进行排序：

```cpp
struct indirect_less {
    template <typename T>
    bool operator()(const T &x, const T &y)
        {return *x < *y;}
};
// ...
std::vector<std::unique_ptr<A>> v;
// ...
std::sort(v.begin(), v.end(), indirect_less());
```

当`sort`移动`unique_ptr`对象时，它会用`swap`（不要求对象可拷贝）或移动构造（赋值）操作。因此，整个算法过程中，每一项都由一个智能指针拥有且引用，该智能指针保持不变。如果算法尝试拷贝操作，会产生编译错误。
