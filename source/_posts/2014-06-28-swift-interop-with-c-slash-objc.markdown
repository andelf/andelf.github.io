---
layout: post
title: "Swift Interop with C/ObjC Part 3 (Swift 与 ObjC 和 C 的交互，第三部分）"
date: 2014-06-28 21:58:35 +0800
comments: true
categories: swift swift-interop
---

声明： 转载请注明，方便的情况下请知会本人. [weibo](http://weibo.com/234632333)

之前说那是最后一篇。可惜越来越发现有很多东西还没介绍到。事不过三。再坑一篇。

## 前言

本文解决如下问题

- ObjC/C 中定义的某个类型、结构体，通过 Bridge Header 或者 Module 对应到 Swift 到底是什么类型
- 指针间的转换问题

补充之前没解决的一些问题，比如提到 ``CMutablePointer`` 的 ``sizeof`` 是两个字长，那么在函数调用中是如何对应到 C 的指针的？

预备内容：

- [Swift 与 Objective-C 之间的交互](http://andelf.github.io/blog/2014/06/11/swift-and-objectivec-interop/)
- [简析Swift和C的交互](http://andelf.github.io/blog/2014/06/15/swift-and-c-interop/)
- [简析 Swift 和 C 的交互，Part 二](http://andelf.github.io/blog/2014/06/18/swift-and-c-interop-cont/)
- [Swift NSError Internals（解析 Swift 对 NSError 操作）](http://andelf.github.io/blog/2014/06/16/swift-nserror-internals/)
- [Swift 的隐式类型转换](http://andelf.github.io/blog/2014/06/08/swift-implicit-type-cast/)
- [Swift Attributes](http://andelf.github.io/blog/2014/06/06/swift-attributes/)

## C/ObjC to Swift 对应规则

以下内容均适合 Objective-C 。第一部分适合 C 。

**以下内容再 Xcode6-beta3 中不适用** 请参考 [Swift 在 Xcode6-beta3 中的变化](http://andelf.github.io/blog/2014/07/08/swift-beta3-changes/)。

### for C

#### 可导出的类型定义

函数、枚举、结构体、常量定义、宏定义。

结构体定义支持：

```c
typedef struct Name {...} Name;
typedef struct Name_t {...} Name;
struct Name { ... };
```

其中无法处理的结构体、函数类型、 varargs 定义不导出。预计以后版本会修复。带 bit field 的结构体也无法识别。

#### 类型对应关系

仔细分析发现，诡异情况还很多。基础类型请参考上几篇。

在函数定义参数中：

类型 | 对应为
:------:|:-------:
``void *`` | ``CMutableVoidPointer``
``Type *``、``Type[]`` | ``CMutablePointer<Type>``
``const char *`` | ``CString``
``const Type *`` | ``CConstPointer<Type>``
``const void *`` | ``CConstVoidPointer``

在函数返回、结构体字段中：

类型 | 对应为
:------:|:-------:
``const char *`` | ``CString``
``Type *``、``const Type *`` | ``UnsafePointer<Type>``
``void *``、``const void *`` | ``COpaquePointer``
无法识别的结构指针  | ``COpaquePointer``

另外还有如下情况：

全局变量、全局常量(``const``)、宏定义常量(``#define``) 均使用 ``var``，常量不带 ``set``。

结构体中的数组，对应为元祖，例如 ``int data[2]`` 对应为 ``(CInt, CInt)``，所以也许。。会很长。数组有多少元素就是几元祖。

### for ObjC

ObjC 明显情况要好的多，官方文档也很详细。

除了 ``NSError **`` 转为 ``NSErrorPointer`` 外，需要注意的就是：

函数参数、返回中的 ``NSString *`` 被替换为 ``String!``、``NSArray *`` 被替换为 ``AnyObject[]!``。

而全局变量、常量的 ``NSString *`` 不变。

## 关于 ``CMutablePointer`` 的行为

上回说到 ``CMutablePointer``、``CConstPointer``、``CMutableVoidPointer``、``CConstVoidPointer``
四个指针类型的字长是 2，也就是说，不可以直接对应为 C 中的指针。但是前面说类型对应关系的时候， C 函数声明转为 Swift
时候又用到了这些类型，所以看起来自相矛盾。仔细分析了 lldb 反汇编代码后发现，有如下隐藏行为:

### in Swift

在纯 Swift 环境下，函数定义等等、这些类型字长都为 2，不会有任何意外情况出现。

### in C/ObjC

当一个函数的声明是由 Bridge Header 或者 LLVM Module 隐式转换而来，且用到了这四个指针类型，那么代码编译过程中类型转换规则、隐式转换调用等规则依然有效。只不过在代码最生成一步，会插入以下私有函数调用之一：

```scala
@transparent func _convertCMutablePointerToUnsafePointer<T>(p: CMutablePointer<T>) -> UnsafePointer<T>
@transparent func _convertCConstPointerToUnsafePointer<T>(p: CConstPointer<T>) -> UnsafePointer<T>
@transparent func _convertCMutableVoidPointerToCOpaquePointer(p: CMutableVoidPointer) -> COpaquePointer
@transparent func _convertCConstVoidPointerToCOpaquePointer(p: CConstVoidPointer) -> COpaquePointer
```

这个过程是背后隐藏的。然后将转换的结果传参给对应的 C/ObjC 函数。实现了指针类型字长正确、一致。

### 结论

作为程序员，需要保证调用 C 函数的时候类型一致。如果有特殊需求重新声明了对应的 C 函数，那么以上规则不起作用，所以重声明 C 中的函数时表示指针不可以使用这四个指针类型。

## 再说指针

{% img left /images/attachments/swift-pointers.png 'Swift Pointers' %}

虚线表示直接隐式类型转换。其中 ``UnsafePointer<T>`` 可以通过用其他任何指针调用构造函数获得。

``CMutablePointer<T>`` 和 ``CMutableVoidPointer`` 也可以通过 ``Array<T>`` 的引用隐式类型转换获得 （ ``&arr`` ）。

椭圆表示类型 ``sizeof`` 为字长，可以用于声明 C 函数。

四大指针可以用 ``withUnsafePointer`` 操作。转换为 ``UnsafePointer<T>``。上一节提到的私有转换函数请不要使用。

## 字符串

之前的文章已经介绍过怎么从 ``CString`` 获取 ``String`` （静态方法 ``String.fromCString``）。

从 ``String`` 获取 ``CString`` 也说过， 是用 ``withCString``。

也可以从 ``CString(UnsafePointer.alloc(100))`` 来分配空数组。

## 参考

- [我的 Github andelf/Defines-Swift](https://github.com/andelf/Defines-Swift)

