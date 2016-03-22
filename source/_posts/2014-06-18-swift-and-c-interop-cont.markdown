---
layout: post
title: "Swift and C Interop Cont. (简析 Swift 和 C 的交互，Part 二)"
date: 2014-06-18 23:38:01 +0800
comments: true
categories: swift swift-interop
---

声明： 转载注明我或者 SwiftChina . [weibo](http://weibo.com/234632333)

本文的发现基于个人研究，目测是官方外的首创。请尊重原创。

本文是讲述 Swift 与 C 交互操作系列文章的第二部分。解决之前的所有遗留问题。

第一部分请参考 [Swift and C Interop 简析Swift和C的交互](http://andelf.github.io/blog/2014/06/15/swift-and-c-interop/)

本文将介绍实际应用过程中会遇到的各类情况。

## 再看类型对应

标准类型这里就不提了，上面的文章讲的很明白了。

### 7 种指针类型

从代码看，我认为 Swift 对应 C 的指针时候，存在一个最原始的类型 ``RawPointer``，但是它是内部表示，不可以直接使用。所以略过。但它是基础，可以认为它相当于 ``Word`` 类型（机器字长）。

**以下内容再 Xcode6-beta3 中不适用** 请参考 [Swift 在 Xcode6-beta3 中的变化](http://andelf.github.io/blog/2014/07/08/swift-beta3-changes/)。

#### COpaquePointer

不透明指针。之前我以为它很少会用到，不过现在看来想错了，虽然类型不安全，但是很多场合只能用它。它是直接对应 ``RawPointer`` 的。字长相等。

> In computer programming, an opaque pointer is a special case of an opaque data type, a datatype declared to be a pointer to a record or data structure of some unspecified type. - 来自 Wikipedia

几乎没有任何操作方法，不带类型，主要用于 Bridging Header 中表示 C 中的复杂结构指针

比如一个例子， libcurl 中的 ``CURL *`` 的处理，其实就是对应为 ``COpaquePointer``。

#### UnsafePointer<T>

泛型指针。直接对应 ``RawPointer``。字长相等。

处理指针的主力类型。常量中的 ``C_ARGV`` 的类型也是它 ``UnsafePointer<CString>``。

支持大量操作方法：

- 通过 ``.memory`` 属性 ``{ get set }`` 操作指针指向的内容
- 支持 subscript ，直接对应于 C 的数组，例如 ``C_ARGV[1]``
- 通过 ``alloc(num: Int)`` 分配数组空间
- ``initialize(val: T)`` 直接初始化
- offset 操作 ``.succ()`` ``.pred()``
- 可以从任意一种指针直接调用构造函数获得
- 隐式类型转换为非 ``COpaquePointer`` 之外的任意一种指针

#### AutoreleasingUnsafePointer<T>

之前特地写文介绍过这个指针类型。``NSError`` 的处理就主要用它。传送门： [Swift NSError Internals（解析 Swift 对 NSError 操作）](http://andelf.github.io/blog/2014/06/16/swift-nserror-internals/)

内部实现用了语言内置特性，从名字也可以看出来，这个应该是非常棒的一个指针，可以帮助管理内存，逼格也高。内存直接对应 ``RawPointer`` 可以传递给 C 函数。

- 通过 ``.memory`` 属性 ``{ get set }`` 操作指针指向的内容
- 直接从 ``&T`` 类型获得，使用方法比较诡异，建议参考文章

#### CMutablePointer<T> CConstPointer<T>

分别对应于 C 中的 ``T *``、``const T *``。不可直接传递给 C 函数，因为表示结构里还有一个 ``owner`` 域，应该是用来自动管理生命周期的。``sizeof`` 操作返回 16。但是可以有隐式类型转换。

操作方法主要是 ``func withUnsafePointer<U>(f: UnsafePointer<T> -> U) -> U``，用 Trailing Closure 语法非常方便。

#### CMutableVoidPointer CConstVoidPointer

分别对应于 C 中的 ``void *``、``const void *``。其他内容同上一种。

#### 小结指针

以上 7 种指针类型可以分未两类，我给他们起名为 第一类指针 和 第二类指针 。（你看我在黑马克思耶，算了这个梗太深，参考马克思主义政治经济学）

- 可以直接用于 C 函数声明的 第一类指针
  - ``COpaquePointer`` ``UnsafePointer<T>`` ``AutoreleasingUnsafePointer<T>``
  - 是对 ``RawPointer`` 的封装，直接对应于 C 的指针，它们的 ``sizeof`` 都是单位字长
- 不可用于声明 第二类指针
  - ``CMutablePointer<T> CConstPointer<T> CMutableVoidPointer CConstVoidPointer``
  - 直接从 Swift 对象的引用获得（一个隐藏特性，引用隐式转换）（主要构造方法）
  - 包含了一个 ``owner`` 字段，可以管理生命周期，理论上在 Swift 中使用
  - 通过 ``.withUnsafePointer`` 方法调用

所有指针都实现了 ``LogicValue`` 协议，可以直接 ``if a_pointer`` 判断是否为 ``NULL``。

``nil`` 类型实现了到所有指针类型的隐式类型转换，等价于 C 中的 ``NULL`，可以直接判断。

什么时候用什么？这个问题我也在考虑中，以下是我的建议。

- 对应复杂结构体，不操作结构体字段的： ``COpaquePointer`` 例如 ``CURL *``
- 日常操作： ``UnsafePointer<T>``
- 同时需要在 Swift 和 C 中操作结构体字段，由 Swift 管理内存：``AutoreleasingUnsafePointer<T>``
- Swift 中创建对象，传递给 C： 第二类指针


## 工具类型

### CVarArg CVaListPointer VaListBuilder

用于处理 C 语言中的可变参数 valist 函数。

```scala
protocol CVarArg {
  func encode() -> Word[]
}
```

表示该类型可以作为可变参数，相当多的类型都实现了这个。

```scala
struct CVaListPointer {
    var value: UnsafePointer<Void>
    init(fromUnsafePointer from: UnsafePointer<Void>)
    @conversion func __conversion() -> CMutableVoidPointer
}
```

对应于 C，直接给 C 函数传递，声明、定义时使用。

```scala
class VaListBuilder {
    init()
    func append(arg: CVarArg)
    func va_list() -> CVaListPointer
}
```

工具类，方便地创建 ``CVaListPointer``。

还有一些工具函数：

```scala
func getVaList(args: CVarArg[]) -> CVaListPointer
func withVaList<R>(args: CVarArg[], f: (CVaListPointer) -> R) -> R
func withVaList<R>(builder: VaListBuilder, f: (CVaListPointer) -> R) -> R
```
 
非常方便。

### UnsafeArray<T>

```scala
struct UnsafeArray<T> : Collection, Generator {
  var startIndex: Int { get }
  var endIndex: Int { get }
  subscript (i: Int) -> T { get }
  init(start: UnsafePointer<T>, length: Int)
  func next() -> T?
  func generate() -> UnsafeArray<T>
}
```

处理 C 数组的工具类型，可以直接 for-in 处理。当然，只读的，略可惜。

### Unmanaged<T>

```scala
struct Unmanaged<T> {
  var _value: T
  init(_private: T)
  func fromOpaque(value: COpaquePointer) -> Unmanaged<T>
  func toOpaque() -> COpaquePointer
  static func passRetained(value: T) -> Unmanaged<T>
  static func passUnretained(value: T) -> Unmanaged<T>
  func takeUnretainedValue() -> T
  func takeRetainedValue() -> T
  func retain() -> Unmanaged<T>
  func release()
  func autorelease() -> Unmanaged<T>
}
```

顾名思义，手动管理 RC 的。避免 Swift 插入的 ARC 代码影响程序逻辑。

## C 头文件的导入行为

### 宏定义

数字常量 CInt, CDouble (带类型后缀则为对应类型，如 1.0f ）
字符常量 CString
其他宏 展开后，无定义

### 枚举 enum

创建 enum 类型，并继承自 ``CUnsignedInt`` 或 ``CInt`` （enum 是否有负初始值）

可以通过 ``.value`` 访问。

### 结构体 struct

创建 struct 类型，只有默认 init ，需要加上所有结构体字段名创建。

### 可变参数函数

转为 ``CVaListPointer``。手动重声明更好。这里举 ``Darwin`` 模块的例子说。

```
func vprintf(_: CString, _: CVaListPointer) -> CInt
```

## 从 C 调用 Swift

只能调用函数。

之前说过，用 ``@asmname("name")`` 指定 mangled name 即可。

然后 C 语言中人工声明下函数。很可惜自动导出头文件不适用于 C 语言，只适用于 Objective-C 。

目测暂时无法导出结构体，因为 Swift 闭源不提供相关头文件。靠猜有风险。

全局变量不支持用 ``@asmname("name")`` 控制导出符号名。目测可以尝试用 mangled name 访问，但是很不方便。

## 示例

我尝试调用了下 libcurl 。

项目地址在 [andelf/curl-swift](https://github.com/andelf/curl-swift) 包含编译脚本（就一句命令）。

Bridging Header 只写入 ``#include<curl/curl.h>`` 即可。

```scala
@asmname("curl_easy_setopt") func curl_easy_setopt(curl: COpaquePointer, option: CURLoption, param: CString) -> CURLcode
@asmname("curl_easy_setopt") func curl_easy_setopt(curl: COpaquePointer, option: CURLoption, param: CBool) -> CURLcode

let handle = curl_easy_init()

// this should be a const c string. curl_easy_perform() will use this.
let url: CString = "http://www.baidu.com"

curl_easy_setopt(handle, CURLOPT_URL, url)
curl_easy_setopt(handle, CURLOPT_VERBOSE, true)

let ret = curl_easy_perform(handle)
let error = curl_easy_strerror(ret)
println("error = \(error)")
```

值得注意的是其中对单个函数的多态声明， ``curl_easy_setopt`` 实际上第三个参数是 ``void *``。

以及对 ``url`` 的处理，实际上 libcurl 要求设置的 url 参数一直保持到 ``curl_easy_perform`` 时，所以这里用 ``withUnsafePointer`` 或者
``withCString`` 是不太可取的方法。实际上或许可以用 ``Unmanaged<T>`` 来解决。

## 总结

我觉得说这么多。。。

调用 C 已经再没有别的内容可说了。其他的就是编程经验的问题，比如如何实现 C 回调 Swift 或者 Swift 回调 C 。可以参考其他语言的做法。解决方法不只一种。

## 参考文献

- [我的 Github andelf/Defines-Swift](https://github.com/andelf/Defines-Swift)。
- [Swift and C Interop 简析Swift和C的交互 第一部分](http://andelf.github.io/blog/2014/06/15/swift-and-c-interop/)
- [Swift NSError Internals（解析 Swift 对 NSError 操作）](http://andelf.github.io/blog/2014/06/16/swift-nserror-internals/)
