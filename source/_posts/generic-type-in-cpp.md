---
title: 用 C++ 实现一个简单的泛型
author: 初時雨
email: Hatsushigure_c@163.com
readmore: true
hideTime: false
date: 2023-07-22 21:54:34
tags: [C plus plus, 模板, 泛型]
description: 第一次写模板, 还是有些不顺利呢
categories: 编程
---
## 什么是泛型

这里的泛型, 指的是一种可以存储任意数据的类型。类似于 C++ 标准库中的 [`any`](https://zh.cppreference.com/w/cpp/utility/any) 和 [`variant`](https://zh.cppreference.com/w/cpp/utility/variant), 但在使用上更像 Qt 的 `QVariant`。使用泛型, 可以方便地用一个容器存储多种类型的数据。我最熟悉的使用泛型的例子是 Qt 的属性系统。很显然, 泛型为代码的编写带来了极大的便利。
## HOW-TO

### 数据存储方案

为了存储各种类型, 我们的类必须以指针的形式来存放数据。因为只有用 `void*` 才可以使类的定义(注意不是成员函数的定义)彻底摆脱模板。至于为什么要摆脱模板? 下图做出了解释。
![](https://i2.100024.xyz/2023/07/22/10o37lr.webp
)
注意到, 模板类的不同实例化属于不同类型, 不符合一开始的要求。

于是这个类可以写成
```cpp
class Variant
{
private:
    void* m_data;
};
```
### 构造函数与析构函数

最简单的是默认构造函数, 由于没有数据, 直接给个空指针就行
```cpp
Variant() : m_data {nullptr} {}
```

这个类使用了堆空间, 不要忘了析构函数释放资源
```cpp
//Variant.h
~Variant()

//Variant.cpp
Variant::~Variant()
{
    delete m_data;
}
```

然后是初始化时指定值了。这时, 我们不得不用上模板。
```cpp
//Variant.h
template <typename T>
Variant(T value)
{
    m_data = new T(std::forward<T>(value));
}
```
虽然这里使用了模板, 可整个类并不属于模板类。这是非常巧妙的处理。此外, 这里还使用了 C++ 11 的完美转发 [`forward`](https://zh.cppreference.com/w/cpp/utility/forward), 以尽可能降低开销, 同时确保不会因为临时变量而出现问题。

很有意思的一点: 在这个函数中使用 C++ 20 添加的 auto 模板形式时不行的, 会引发一个编译器内部错误。如图。
![](https://i2.100024.xyz/2023/07/22/116j7ym.webp)
就很怪🤔🤔🤔

接下来是另外两个特殊的构造函数。由于我们并没有存储任何关于 `m_data` 类型的信息, 复制 `m_data` 似乎不太可能。故删除拷贝构造函数。
```cpp
Variant(Variant&) = delete;
```

移动构造函数只需要转移数据的所有权, 而无需真正复制数据。
```cpp
//Variant.h
Variant(Varinat&& other) noexcept;

//Variant.cpp
Variant::Variant(Variant&& other) noexcept
{
    m_data = other.m_data;
    other.m_data = nullptr;
}
```
### 等号赋值函数

和前面的大同小异, 同样要注意删掉传入 `Variant&` 的。

上代码
```cpp
//Variant.h
template <typename T>
void operator=(T value) { m_data = new T(std::forward<T>(value)); }
void operator=(Variant&) = delete;
void operator=(Variant&&) noexcept;

//Variant.cpp
void Variant::operator=(Variant&& value) noexcept
{
	m_data = value.m_data;
	value.m_data = nullptr;
}
```
### 其他函数
写到这里, 我们已经可以创建这个类的对象了, 但并没有方法拿到它真实的值。下面就来解决这个问题。

~~知乎上有句名言:「先问是不是, 再问为什么」~~。类似地, 我们先要知道 `Variant` 有没有值, 再来获取它的值。
```cpp
bool hasValue() const { return (nullptr != m_data); }
```
这个也很好理解, 把 `nullptr` 写前面也是为了防止不小心修改成员的值, 和 `const` 一起相当于双重保险了。

最重要的部分——「转换函数」要来了。打引号的原因是它并非真正的转换函数, 不过功能相同。先上代码
```cpp
template<typename T>
const T& Variant::value() const
{
	if (!hasValue())
		return T();
	return *static_cast<T*>(m_data);
}
```
逐步解析一下。

首先, 它的返回值为什么是 `const` 引用? ~~因为不能是非 `const` (笑)~~。去掉 `const` 后, 编译报错: 非常量引用只能绑定到左值。因此只能加 `const`, 后续若需修改, 可以使用 [`const_cast`](https://zh.cppreference.com/w/cpp/language/const_cast) 去掉 `const`。

然后, 是一个判断。其实我本来想用 `throw` 的, 但已经不想增加代码量了, 于是创建了一个空对象。

最后一行, 就是把 `m_data` 转换为合适的类型后解引用, 这样就能得到所需的值了。
## 反思

这次的 `demo`, 和标准库的实现还有很大的差距, 体现了我的诸多不足。如在引用、值类别、模板方面还有很大的欠缺, 今后还要再接再厉。~~俺も頑張らないと！~~
