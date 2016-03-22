---
layout: post
title: "Swift NSError Internals（解析 Swift 对 NSError 操作）"
date: 2014-06-16 14:45:15 +0800
comments: true
categories: swift
---

声明： 原创发现，原创内容。转载注明我或者 SwiftChina . [weibo](http://weibo.com/234632333)

其中基础部分借鉴了 WWDC 407 部分的介绍。

相关的分析主要基于我 dump 出的 Swift 标准库声明代码，位于 [我的 Github andelf/Defines-Swift](https://github.com/andelf/Defines-Swift)。

## 引子

WWDC 407 中简单提到了 NSError 在 Swift 中的处理方法，说``NSErrorPointer is Swift’s version of NSError **``，其中大概内容是：

- Foundation 的所有函数中的 ``NSError **`` 都被对应为 Swift 中的 ``NSErrorPointer`` (自定义函数也应如此)
- 调用时候使用下面的方式：
```scala
var error: NSError?
foobar(...., error: &error)
```
- 这里一定要用 ``var``，只有 var 才有引用的概念，才可以作为左值
- 定义时（重载一些带 NSError 参数的函数）(注意其中的 ``.memory`` 部分。)：
```scala
override func contentsForType(typeName: String!, error: NSErrorPointer)  -> AnyObject! { 
    if cannotProduceContentsForType(typeName) {  if error { 
        error.memory = NSError(domain: domain, code: code, userInfo: [:])  } 
    return nil  } 
    // ...
}
```

到这里，基本怎么在 Swift 使用 ``NSError`` 已经足够了。但是本着探究妈蛋 Swift 到底藏起了多少黑魔法的伟(er)大(bi)精神，我继续分析下这里面的槽点。

## 这不科学！

我们知道 Swift 是强类型语言。类型不同怎么能在一起？直接出错才对。

当然也可能是之前提到过的“隐式类型转换”，所以这里探索下 ``NSError`` 和 ``NSErrorPointer`` 的关系，到底是如何实现了 Objective-C 中的 ``NSError **``。

而且有一个逆天的地方是，``NSErrorPointer`` 没有被标记为 ``inout``，但传递参数时候需要传递 ``NSError`` 的引用。

## 上代码

这里给出 Swift 中 ``NSError``、``NSErrorPointer`` 及相关类型的声明代码。(部分）

```scala
@objc(NSError) class NSError : NSObject, NSCopying, NSSecureCoding, NSCoding {
  @objc(initWithDomain:code:userInfo:) init(domain: String!, code: Int, userInfo dict: NSDictionary!)
  @objc(errorWithDomain:code:userInfo:) class func errorWithDomain(domain: String!, code: Int, userInfo dict: NSDictionary!) -> Self!
  @objc var domain: String! {
    @objc(domain) get {}
  }
  @objc(init) convenience init()  
  ....
}

typealias NSErrorPointer = AutoreleasingUnsafePointer<NSError?>

struct AutoreleasingUnsafePointer<T> : Equatable, LogicValue {
  let value: RawPointer
  @transparent init(_ value: RawPointer)
  @transparent static func __writeback_conversion_get(x: T) -> RawPointer
  @transparent static func __writeback_conversion_set(x: RawPointer) -> T
  @transparent static func __writeback_conversion(inout autoreleasingTemp: RawPointer) -> AutoreleasingUnsafePointer<T>
  var _isNull: Bool {
    @transparent get {}
  }
  @transparent func getLogicValue() -> Bool
  var memory: T {
    @transparent get {}
    @transparent set {}
  }
}
```

## 分析

由以上代码可以知道，``NSErrorPointer`` 实际上是 ``AutoreleasingUnsafePointer<Optional<NSError>>`` 类型（这里我顺便展开了 ``Type?``），
``AutoreleasingUnsafePointer<T>`` 封装了一个 ``RawPointer``，发挥想象力，差不多知道它就是个 ``void *``，指向类型 ``T``。在本例中，指向 ``NSError?``。

一般情况下 Swift 的 ``NSError!`` 被用于对应 Objective-C 的 ``NSError *``， 一层指针，可 nil. 所以这里，两层指针算是对应上了（一层 RawPointer, 一层靠语言本身的特性实现，这里暂时不考虑 ``Type!`` 和 ``Type?`` 的差异性，使用时候多注意就可以）。

然后就是棘手的问题，这里的类型转换是如何实现的？这里没有看到标准的隐式类型转换。但是有 ``__writeback_conversion*`` 一系列 static 函数。应该是他们起到了转换的作用（再次说明，Swift 是强类型语言，所以必然这样的特性有黑魔法）。

从名字和标准库的定义看，这三个函数应该是同时起作用，用于表述三个类型的关联关系。我写了个例子。

### 示例代码

```scala
struct Foobar {
    var val: Int

    @transparent static func __writeback_conversion_get(x: String) -> Int {
        println("location 1")
        if let v = x.toInt() {
            return v
        } else {
            return 0
        }

    }
    @transparent static func __writeback_conversion_set(x: Int) -> String {
        println("location 2")
        return toString(x + 10000)
    }
    @transparent static func __writeback_conversion(inout temp: Int) -> Foobar {
        println("location 3")
        return Foobar(val: temp)
    }

}

func test_foobar(v: Foobar) {
    dump(v, name: "dump from test_foobar")
}

var word = "9527"
test_foobar(&word)
dump(word, name: "word")
```

如上，代码输出是：

```
location 1
location 3
▿ dump from test_foobar: V5trans6Foobar (has 1 child)
  - val: 9527
location 2
- word: 19527
```

这三个函数在一次隐式调用中，全部都用到了，调用顺序如上。看起来隐藏在背后的关系明了了一些，流程大概是：

- 用 ``word`` 调用 ``Foobar.__writeback_conversion_get``，获得一个 ``Int``
- 用 ``Int`` 调用 ``Foobar.__writeback_conversion``，构造了一个 ``Foobar`` 对象
- 将 ``Foobar`` 交给 ``test_foobar`` 函数内部处理
- 函数逻辑结束后，调用 ``Foobar.__writeback_conversion_set``，获得字符串，赋值给 ``word``

我返回到 ``NSError`` 说。

### 回到 NSError 说

最简删节版代码：

```scala
// T = NSError? = Optional<NSError>
struct AutoreleasingUnsafePointer<T> {
  let value: RawPointer
  static func __writeback_conversion_get(x: T) -> RawPointer
  static func __writeback_conversion_set(x: RawPointer) -> T
  static func __writeback_conversion(inout autoreleasingTemp: RawPointer) -> AutoreleasingUnsafePointer<T>
  ...
  var memory: T { get set }
}  
```

当一个函数接受 ``AutoreleasingUnsafePointer<T>`` 作为参数的时候，我们可以直接传递 T 的引用 ``&T``，然后整个流程自动转换为

- 用该引用参数调用 ``__writeback_conversion_get`` 获得一个 ``RawPointer``
- 调用 ``__writeback_conversion`` 获得一个 ``AutoreleasingUnsafePointer<T>`` 对象
- 将这个对象交给函数内部处里，调用相关方法
- 函数返回时，调用 ``__writeback_conversion_set``，然后将结果赋值给一开始的引用参数，类型 ``&T`` 的那个。


如果是 ``NSError`` 的情况, 情况如下：

- 某库函数接受 ``NSErrorPointer`` 作为参数，我们直接传递 ``NSError?`` 的引用 ``&error``
- ``error`` 作为参数调用 ``__writeback_conversion_get``， 获得 ``RawPointer``
- 用这个 ``RawPointer`` 调用 ``__writeback_conversion``，获得一个 ``NSErrorPointer`` 对象
- 函数进行相关处理。对于 NSError 来说，这里一般是 Foundation 代码或者是 override 后的代码，在出错或者异常时访问 ``.memory``，设置相应的错误
- 函数返回。用之前的 ``RawPointer`` 调用 ``__writeback_conversion_set``，获得 ``NSError?`` 然后赋值给引用 error
- 接下来代码中可以对 error 进行判断，处理异常

其中 ``RawPointer`` 之前我们已经讨论过，是指针，在整个过程中值不变，指针指向的内存，就是 ``.memory`` 属性操作的部分，才是可变的部分，不属于 ``AutoreleasingUnsafePointer<T>`` 的成员访问控制范围，所以这就是为什么函数参数中的 ``NSErrorPointer`` 为什么没有标记为 ``inout`` 的原因。

## 总结

这三个 static 函数合起来组成了整体功能。我觉得可以把这个功能叫做“引用的隐式类型转换”或者"隐式引用类型转换"。

当然，它的强大之处在于用隐藏了二重指针的细节，同时这个特性也可以有其他作用。比如。。。（我没想好）

## 参考文献

- WWDC 407
- [我的 Github andelf/Defines-Swift](https://github.com/andelf/Defines-Swift)
- [隐式类型转换](http://andelf.github.io/blog/2014/06/08/swift-implicit-type-cast/)
