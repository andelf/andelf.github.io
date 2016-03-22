---
layout: post
title: "Implicit Type Convertion in Swift (Swift 的隐式类型转换)"
date: 2014-06-08 00:25:43 +0800
comments: true
categories: swift
---

这是我自己给的命名，因为这个特性暂时看是 undocumented 的。通过 lldb 和 Xcode 的变量 inspector 功能发掘出来的。然后尝试调用没有问题。

This is an undocumented feature. I found it by lldb and the variable inspector of Xcode6-beta.

形式: ``@conversion func __conversion<T>() -> T { }``

对了 Reddit 网友提出 ``@conversion`` 是可以省略的。个人认为编译器可能对不同的两种方式行为略有不同，最好还是加上。还有就是 ``@transparent`` 其实是表示 inline 。

先随便设想个场合。比如我们某个库需要表示一个二维点。

Suppose we need a Point type in our library.

```scala
struct Point {
    var x: Int = 0
    var y: Int = 0
}
let p = Point(x: 23, y: 45)
```

相信看了几天的 Swift，信手捏来啊。啧啧。
然后你发现你的另个库已经用了 (Int, Int) 来表示二维点了。。。。。你的把手头一堆的 Point 在转成 (Int, Int)，或者加个什么方法。？

For some reason, we need to work with another library, which uses ``(Int, Int)`` as the representation of a point.
Must we refacotr the code and add type conversion function calls?

妈蛋重构么？

No no~~

我们这样来

复习下 extension 语法, 加上这里要介绍的“隐式类型转换”

We can use the Swift's hiding feature -- Implicit Type Convertion(or to say Cast?).


```scala
extension Point {
    @conversion func __conversion() -> (Int, Int) {
        return (x, y)
    }
}
```

赞爆了。然后我们就可以这么写

The we can directly make a ``(Int, Int)`` from a ``Point``.

```scala
let (x, y) = p
// suppose: func drawPoint(pt: (Int,Int)) { }
drawPoint(p)
```

你看无痛苦吧。然后所有需要 (Int, Int) 参数的函数，直接传递 Point 类型是没有任何问题的。（这个我也很吃惊，理论上这个玩意还是有个推导过程的，会有BUG什么的，没想到老crash 的 Swift 编译器竟然这个做得很好)

好吧这个例子实在是太渣了。我们换个例子说。

假设，我们有个库函数，接受 Color struct (r, g, b 三原色表示) 作为参数。
比如调用  ``setColor(Color(r:23, g:34, b:34))``
然后实际上颜色我们用字符串表示，比如"white", "black" 或者 UInt32 表示比如 #ffeeff 。。。

然后为了方便，我们本来需要写一些转换函数，然后调用。

现在不同了，比如我们给 String 写个 隐式转换，代码里直接用。（用protocol 也可以实现相同的功能）

```scala
extension String {
    @conversion func __conversion() -> Color {
        switch self {
        case "red":
            return Color(r:255, g:0, b: 0)
        case "green":
            return Color(r:0, g:255, b:0)
        .....
        default:
            return Color(r:0, g:0, b:0)
        }
    }
}
```

然后代码里用到 Color 的地方你直接写个 "red" 啊 "pink" 骚粉啊。都是没有问题的。
setColor("red")

上面都是废话。

隐式类型转换标准： ``@conversion func __conversion<T>() -> T { }`` 可以作为方法写到 struct 里。也可以通过 extension 附加。

**以下内容再 Xcode6-beta3 中不适用** 请参考 [Swift 在 Xcode6-beta3 中的变化](http://andelf.github.io/blog/2014/07/08/swift-beta3-changes/)。

然后就是语言本身中有个细小的问题 微博 onevcat 文章里提到。

nil 是 NilType 的唯一实例（来自Python的同学对这种定义一定不陌生)
而 Type? 是 Optional<Type> 类型的，是个enum，变种是  Some(T) 和 None.

```scala
enum Optional<T>  {
    case None
    case Some(T)
}
```

那同学你有没有想到 ，我们为什么可以给一个 Type? 赋值 nil 呢?

答案就是这里讲到的 隐式类型转换。

In Swift standard library, the unique ``nil`` has a type of ``NilType`` and is the only instance. And ``Type?`` is the
shortcut to ``Option<T>``， an enum.

Why we can directly set a ``Type?`` variable to ``nil``?

The Implicit Types Convertion is the answer. The ``NilType`` implements conversion methed to ``Type?`` and ``Type!``.

I've dump the definitions from lldb. You can find them in:

我导出的一些标准库定义在下面的链接。

[My Github Repo](https://github.com/andelf/Defines-Swift/blob/master/Swift-misc.swift)
