---
title: const 在 C 与 C++ 中的异同
author: 初時雨
email: Hatsushigure_c@163.com
readmore: true
hideTime: false
date: 2024-01-13 23:38:32
tags:
description:
categories:
---

好久不见啊，时隔将近三个月，~~某人终于想起来她还有个博客了~~。这是 2024 年的第一篇技术类文章（~~当然希望不是最后一篇~~）。这里简单聊一聊 `const` 关键字在 C 语言和 C++ 中的相同和不同，也算是给以后的自己填坑了。

## 为什么突然想研究这个问题

原因很简单，**踩坑了**。先上代码：

```c
//Utils.h
//...
static const size_t MaxIdentifierLength = 255;
//...

//Statement.h
//...
typedef struct CreateStatement
{
	Statement parent;
	char tableName[MaxIdentifierLength + 1];
	MyVector* fields;	//Vector of FieldInfo
} CreateStatement;
//...
```

这段代码，乍一看好像没有问题对吧，可事实真的如此吗？构建之后，我得到了两个主要的报错（编译失败了）：

![](https://pic.imgdb.cn/item/65a2b79b871b83018ac71d81.jpg)

可是在我的印象中, `const` 是常量，怎么会报错呢？这里就要牵涉到本文的主题了——`const` 在 C 与 C++ 中是不同的！

## 相同点

`const` 在这两门语言中的作用大体上是相同的 (C++ 中关于类的部分不提及)，都是声明修饰对象<font color=red>**不可变**</font>。至于为什么是「**不可变**」而不是「*常量*」，会在后面提及。

例如下面的代码：

```c
const int a = 0;
a = 10; //Illegal
```

显而易见, `a` 是不可变的，尝试给其赋值会导致编译失败。这很符合直觉。此外，还有经典考试八股文，各种指针常量、常量指针，它们虽然难以理解，但事实上是符合直觉的，而这在 C 和 C++ 上是一致的。

## 不同

作为两门*独立*的语言，二者都有的东西并不一定要相同。C 的写法, C++ 会兼容。可是反过来，作为「灵感提供者」的 C 语言从未承诺过兼容 C++。下面就是很好的例子。

### 历史

C 的历史比 C++ 长很多，而在其刚刚面世的时候，其实是没有 `const` 这一关键字的。`const` 是在 ANSI C (亦即 C89) 中才从 C++ 中借鉴而来的。主要用途可以相同，但细节实现上，C 并没有选择让 `const` 取代宏，这才有了二者的区别，

### 常量还是常变量？

对于 C++ 而言, `const` 修饰的常量就是常量。它在任意情况下都不可变，即使尝试用非法手段更改。例如前面的 `const int a = 0;`, `a` 与常量 `10` 的地位是对等的。你无法给 `10` 赋值，自然也不能改变 `a`。也正因为此, C++ 中 `const` 修饰的常量是可以作为数组的长度和索引的，也能作为模板的参数。但在 C 语言中，情况不一样了。即使用 `const` 修饰了，变量依旧是变量。只不过它的值一经设定就无法通过合法的手段修改罢了。

这里文档中用词十分严谨：

> 编译器**可以把**声明为带 `const` 限定类型的对象放到只读内存中

这也就是说，也可以不这么做。因为 C 语言中它是变量，放进只读内存仅仅是优化。

可能你注意到了，我在上一段一直说的都是 **「`const` 修饰的常量」**。那是因为, C++ 中的 `const` 还有一种与 C 语言类似的情况：

```cpp
int a = 0
std::cin >> a;
const int b = a;
int arr[b] {0}; //Illegal!
```

从语法上来看，这是完全可行的，事实上也是如此。但这里的 `b` 退化为了类似 C 语言中的「常变量」，即 `b` 也不能作为数组长度了。原因是在 C++ 中引入了「[常量表达式 (constexpr)](https://zh.cppreference.com/w/cpp/language/constant_expression)」。`b` 是 `const` 的，但其不是常量表达式，因此是一个变量。这里还与编译器优化等内容相关，不再赘述。

最后，看看文档上是怎么说的：

> C 从 C++ 接纳了 `const` 限定符，但不同于 C++，C 中 `const` 限定类型的表达式不是[常量表达式](https://zh.cppreference.com/w/c/language/constant_expression)；它们不可用作 [`case`](https://zh.cppreference.com/w/c/language/switch) 标号，或用于初始化[静态](https://zh.cppreference.com/w/c/language/storage_duration)和[线程](https://zh.cppreference.com/w/c/language/storage_duration)存储期对象，用作[枚举项](https://zh.cppreference.com/w/c/language/enum)，或[位域](https://zh.cppreference.com/w/c/language/bit_field)大小。以之为[数组](https://zh.cppreference.com/w/c/language/array)大小时，产生的数组为 `VLA`。

### 链接性

除了上面这一点区别, C 语言还在链接性上给我们挖了个坑。当然，这也与常**变量**的**变量**本质有关。一般地，全局变量的默认链接性为外部，即所有使用这个名字的全局变量都是同一个，它们共享相同的地址。这就导致了一个问题——我们不能像 C++ 中那样或是像传统的宏定义那样直接全局共享常量了。如果需要，则只能显式地加上 `static` 修饰，让每一个包含该头文件的文件都拥有常量的一个副本，正如 C++ 那样。

> 在未声明为 `extern` 的非局部非异变`非模板 (C++14 起)` `非 inline (C++17 起)`变量声明上使用 `const` 限定符，会给予该变量[内部连接](https://zh.cppreference.com/w/cpp/language/storage_duration#.E8.BF.9E.E6.8E.A5)。这有别于 C，其中 `const` 文件作用域对象拥有外部连接。

## 总结

C 语言与 C++ 中的 `const` 关键字大体上作用是相同的，但仍然有一定区别：

| | C | C++ |
| :--- | :---: | :---: |
| 常量表达式（使用常量初始化）| 否 | 是 |
| 常量表达式（使用变量初始化）| 否 | 否 |
| 链接性 | 外部 | 内部 |

以上，在写代码的时候一定要注意。

## 参考

- [const 类型限定符 - cppreference.com](https://zh.cppreference.com/w/c/language/const)
- [cv（const 与 volatile）类型限定符 - cppreference.com](https://zh.cppreference.com/w/cpp/language/cv)
- [C/C++ 基础中的基础： const 修饰符用法总结！ - 知乎](https://zhuanlan.zhihu.com/p/90720012)
