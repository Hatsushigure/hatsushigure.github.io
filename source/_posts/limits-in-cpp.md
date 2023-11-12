---
title: C++ 中的 limits 头文件
date: 2022-09-20 14:07:59
tags: ["C plus plus", "冷知识"]
categories: 编程
description: 使用新的 limits
---
对于 C++中的整型范围，常用的头文件是`climits`。但除此以外，还有`limits`这个头文件也可以提供相同功能，并且看起来更加「现代」。
## 发现
几天前的CSP-J/S认证中，有这样一段代码:
```cpp
#include <limits>
//...
int main()
{
	//...
	int ret = numeric_limits<int>::max();
	//...
}
```
虽然看着代码很好理解，但我当时确实是第一次见到这个头文件。事后，经过一番查找，我对这个文件的内容有了初步了解...
## 作用
和`climits`相同，用于提供整型的取值范围。
## 使用方法
| | |
| ----- | ----- |
|  numberic_limits<type>::max()  |  最大值  |
|  numberic_limits<type>::min()  |  最小值  |
## 优点(好像没有很大区别来着)
~~没有宏，看起来更舒服~~
功能更加丰富(太复杂了，而且基本用不到，这里不再介绍)，且更加安全。

------------

**END**