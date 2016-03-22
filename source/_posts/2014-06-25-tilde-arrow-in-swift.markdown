---
layout: post
title: "Tilde Arrow in Swift （Swift 标准库中的波浪箭头 ~> ）"
date: 2014-06-25 22:44:29 +0800
comments: true
categories: swift
---

本文瞎写，没实际内容。请不用看了。

## 摘要

本文挖掘 Swift 标准库中的诡异操作符 ``~>`` 波浪箭头的作用。

## 正文

查看标准库定义的时候，发现了一个奇怪的运算符 ``~>``，看起来高大上，所以探索下它到底起什么作用。

标准库对 ``~>`` 用到的地方很多，我取最简单的一个来做说明。

```
protocol SignedNumber : _SignedNumber {
  func -(x: Self) -> Self
  func ~>(_: Self, _: (_Abs, ())) -> Self
}
func ~><T : _SignedNumber>(x: T, _: (_Abs, ())) -> T
func abs(_: CInt) -> CInt
func abs<T : SignedNumber>(x: T) -> T
```

这是对有符号整型的一个协议，我去掉了额外的属性。事实上 ``_Abs`` 类型是一个空结构， ``sizeof`` 为 0 。

写个测试程序，计算下 ``abs(-100)`` 看看情况，发现 ``top_level_code()`` 调用了 ``SignedNumber`` 版本的 ``abs()``：

```
callq  0x100001410               ; Swift.abs <A : Swift.SignedNumber>(A) -> A
```

反汇编这个库函数，发现一个有意思的调用：

```
callq  0x10000302a               ; symbol stub for: Swift._abs <A>(A) -> (Swift._Abs, A)
```

这个 ``_abs()`` 函数是私有函数， Swift 中把很多私有的函数、成员变量、结构、协议都以下划线开头，意思就是不希望我们去调用或者访问的函数，在缺乏成员访问控制的语言中，其实这么做也不错。大家可以借鉴。

而 ``_abs()`` 函数很简单，将任意类型 T 直接封装成 ``(_Abs, T)`` 元组，返回。

然后代码的逻辑就是用这个元祖解开重新组装，调用 ``~>``。逻辑如下：

```scala
// logic of abs() funciton
let (operation, val) = _abs(-100)
val ~> (operation, ()) // 返回 100
```

到这里就清楚了。实际上 ``~>`` 将一个简单的操作复杂化。多调用了层，实际开销主要在元祖的解开和重组装（实际开销理论上在优化模式下应该可以忽略，因为包含 ``_Abs``， size 为 0）。

到这里很多朋友应该已经知道怎么回事了。 ``SignedNumber`` 中的 ``~>`` 操作是为我们提供了一个方法可以 hook 到标准库的 ``abs()`` 函数。来自 Haskell 的同学应该会见过这种单纯地用类型签名来实现函数分发调用的方式。

### 优点？

暂时正在考虑。想明白会发出来。

### 延伸

其实很多标准库函数都用到了类似的方法实现。都用到了 ``~>`` 运算符。包括：

```
countElements()
// _countElements() 工具函数  _CountElements 结构
underestimateCount()
// _underestimateCount() 、 _UnderestimateCount
advance()
// _advance() 、 _Advance
```

等。

## 附录

这里列出部分定义：

```
protocol Sequence : _Sequence_ {
  typealias GeneratorType : Generator
  func generate() -> GeneratorType
  func ~>(_: Self, _: (_UnderestimateCount, ())) -> Int
  func ~><R>(_: Self, _: (_PreprocessingPass, ((Self) -> R))) -> R?
  func ~>(_: Self, _: (_CopyToNativeArrayBuffer, ())) -> ContiguousArrayBuffer<Self.GeneratorType.Element>
}
protocol Collection : _Collection, Sequence {
  subscript (i: Self.IndexType) -> Self.GeneratorType.Element { get }
  func ~>(_: Self, _: (_CountElements, ())) -> Self.IndexType.DistanceType
}

protocol ForwardIndex : _ForwardIndex {
  func ~>(start: Self, _: (_Distance, Self)) -> Self.DistanceType
  func ~>(start: Self, _: (_Advance, Self.DistanceType)) -> Self
  func ~>(start: Self, _: (_Advance, (Self.DistanceType, Self))) -> Self
}
```

相关更多声明代码信息请参考 我的 Github : [andelf/Defines-Swift](https://github.com/andelf/Defines-Swift) 。

## 总结

通过 ``~>`` 和 ``protocol`` 可以自定义编译器的行为。相当于 hook 标准库函数。由于内部实现未知，还不能继续断言它还有什么作用。

但是和直接用 extension 实现协议的方法相比，这个有什么好处呢？待考。

## 更新

可以避免 ``protocol`` 中的静态函数混淆空间，如果用全局函数，那么相当于全局函数去调用静态函数。

还有就是在使用操作符的时候，如果定义多个，那么需要编译器去寻找可用的一个版本。

仔细查看目前的 ``protocol`` 实现，发现还是有点 BUG ，类型限制还是不清楚，表述高阶类型的时候。

为了描述 ``~>`` 的用法，我写了个 [Monad.swift](https://gist.github.com/andelf/6a8432ef0820de9991f6) 。


