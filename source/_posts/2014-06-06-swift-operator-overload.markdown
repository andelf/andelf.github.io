---
layout: post
title: "Swift 运算符重载、自定义运算符"
date: 2014-06-06 10:28:02 +0800
comments: true
categories: swift
---

挖坑先。其实是ibooks书最后的参考手册部分。
这里将手册相关内容都搬运了过来。:)

Swift 对运算符重载的控制很类似 Haskell，而不是 C 系列。如下：

+ 支持自定义运算符 ``/ = - + * % < > ! & | ^ . ~`` 的任意组合。可以脑洞大开创造颜文字。
+ 支持前缀（例如默认的 ``~，！，++, --`` )，后缀（后置的 ``++, --``)，中缀（``* /`` 等等）运算符。
+ 支持自定义优先级。

运算符重载通过函数定义来完成，其实运算符就是一个一元函数，或者是二元函数。

所以那个传的很神的例子 numbers.sort(<) 不算什么，说到底就是个比较函数而已。（来自Haskell的同学基本可以忽略这些内容了，这些太熟悉了）

## 关键字

首先介绍用到的关键字，不得不说，Swift的关键字和Swift前男友一样多。

+ operator 
+ prefix 前缀运算符，比如 （取正+、取负- 、自增++、自减--）
+ postfix 后缀运算符，比如（自增++、自减--），这两可以前缀也可以后缀
+ infix 中缀，最常见二元运算
+ precedence 优先级的意思，取值 0~255 ，纯数字，不可以带符号，下划线，指数e/p 表示
+ associativity 结合性，可以是 left, right, 或者 none

还有用到的属性，就是以 @ 开头的，这个理论上不算是关键字啊。别叫错了。（不过其实无所谓怎么叫，圈里有太多太多的人纠结于简单的定义为一个定义的名字争面红耳赤其实都说的一个意思。名字真的是很次要的东西，用英语吧）

+ @prefix
+ @postfix
+ @infix
+ @assignment 表示运算符具有“赋值”副作用，比如 ``++, +=`` 这样。

先来个最简单的，假设我们有

    struct Point {
        var x: Int
        var y: Int
    }
    let p = Point(x: 100, y: 200)

## 前缀运算符
来个不靠谱的，假设我要执行 -p 这个操作。由于 "-" 已经是定义好了的运算符（减法），直接实现运算即可：

    @prefix func - (pt: Point) -> Point {
        return Point(x: -pt.x, y: -pt.y)
    }
    println(-p) // 看起来高大上

@prefix  属性表示这个 - 运算作为前缀运算符处理，相当于取负。

## 中缀运算符
这么一来，接下来的需求就是两个点之和，“+“ 加法。同理 “+“已经定义，不需要再次定义。直接实现它

    @infix func + (pt0: Point, pt1: Point) -> Point {
        return Point(x: pt0.x + pt1.x, y: pt0.y + pt1.y)
    }

可以在 Playground 里输入 p + p 看看结果。

有了 p + p，直接实现乘法是不是更好呢？乘以一个 Int。这里 "*" 操作的两边就不是同个类型了。需要注意。

    @infix func * (pt: Point, scale: Int) -> Point {
        return Point(x: pt.x * scale, y: pt.y * scale )
    }

输入 p * 3 查看效果，然后输入 3 * p 发现出错，所以运算符左右两边的顺序是很重要的。（来自Python的同学可能会知道 Py 的运算符重载的左右类型是有一个尝试顺序的， ``__mul__, __rmul__`` 等等）

## 后缀运算符
假设，我们的应用里，需要经常计算点到原点(0,0) 的距离，我们“发明”个新运算好了，我们就选一个 "~!"， 来个后缀的，期望 "p~!" 取得点到原点的距离。

    operator postfix ~! {} // 定义新运算符
    @postfix func ~! (pt: Point) -> Double {
        return sqrt(NSNumber.convertFromIntegerLiteral(pt.x * pt.x + pt.y * pt.y).doubleValue)
    }

平方根算得略纠结。。。忽略好了。反正暂时这么可以调用。 然后输入 ``p~!`` 获得距离。

## 自定义中缀运算符
前面基本已经介绍了怎么重载运算符，怎么实现。以及简单介绍了后缀运算符的自定义。

实际上中缀运算符（二元运算）牵扯到的优先级和结合性问题还是挺纠结的。要比前缀后缀复杂得多，当然重载就不涉及，因为是系统默认的优先级。

默认的系统自带的我搬运到后面。

定义中缀运算符首先要制定结合型和优先级。

假设计算两个点之间的距离

    operator infix <~> {
        precedence 250 // 0~255 的一个值，具体多少请参考附录的列表
        associativity left // 或者right, none
    }
    @infix func <~> (pt0: Point, pt1: Point) -> Double {
        return .... // x 之差平方 +  y 之差平方，然后开放
    }

## 带赋值的运算符
前面说了一堆，其实忽略了一些特殊的情况，比如 "+=", "++", 它们的特点是，会修改原有的操作数，有赋值副作用。实际上实现起来大同小异。

    @prefix @assignment func ++ (inout pt: Point) -> Point {
        pt.x += 1
        pt.y += 1
        return pt
    }

这里起作用的是 @assignment 属性，和 inout 关键字。

对于 "+=" 二元操作，第一个操作数需要标记为 inout。类似。大家自己研究好了。


## 附录
从手册里搬运下优先级和结合性：

级别 0~255

+ Exponentiative (No associativity, precedence level 160)
  + ``<< Bitwise left shift``
  + ``>> Bitwise right shift``
+ Multiplicative (Left associative, precedence level 150)
  + ``* Multiply``
  + ``/ Divide``
  + ``% Remainder``
  + ``&* Multiply, ignoring overflow``
  + ``&/ Divide, ignoring overflow``
  + ``&% Remainder, ignoring overflow``
  + ``& Bitwise AND``
+ Additive (Left associative, precedence level 140)
  + ``+ Add``
  + ``- Subtract``
  + ``&+ Add with overflow``
  + ``&- Subtract with overflow``
  + ``| Bitwise OR``
  + ``^ Bitwise XOR``
+ Range (No associativity, precedence level 135)
  + ``.. Half-closed range``
  + ``... Closed range``
+ Cast (No associativity, precedence level 132)
  + ``is Type check``
  + ``as Type cast``
+ Comparative (No associativity, precedence level 130)
  + ``< Less than``
  + ``<= Less than or equal``
  + ``> Greater than``
  + ``>= Greater than or equal``
  + ``== Equal``
  + ``!= Not equal``
  + ``=== Identical``
  + ``!== Not identical``
  + ``~= Pattern match``
+ Conjunctive (Left associative, precedence level 120)
  + ``&& Logical AND``
+ Disjunctive (Left associative, precedence level 110)
  + ``|| Logical OR``
+ Ternary Conditional (Right associative, precedence level 100)
  + ``?: Ternary conditional``
+ Assignment (Right associative, precedence level 90)
  + ``= Assign``
  + ``*= Multiply and assign``
  + ``/= Divide and assign``
  + ``%= Remainder and assign``
  + ``+= Add and assign``
  + ``-= Subtract and assign``
  + ``<<= Left bit shift and assign``
  + ``>>= Right bit shift and assign``
  + ``&= Bitwise AND and assign``
  + ``^= Bitwise XOR and assign``
  + ``|= Bitwise OR and assign``
  + ``&&= Logical AND and assign``
  + ``||= Logical OR and assign``
