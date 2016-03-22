---
layout: post
title: "Swift String Extension for Integer Index(扩展String使之可以支持数字Index和数字Range)"
date: 2014-06-12 11:17:28 +0800
comments: true
categories: swift
---

使用效果:

```scala
let foo: String = "Hello World 世界"
println("foo[13] \(foo[13])")
println("foo[-1] \(foo[-1])")
println("foo[-2] \(foo[-2])")
dump(foo[1..5], name: "foo[1:4]")
dump(foo[10...13], name: "foo[10:13]")
```

简单介绍下实现背后的思路和想法。

首先熟悉 String/Range 的一定知道这玩意 subscript 类型只能是 ``String.Index``，也就是通过 ``String.startIndex`` 或者 ``.endIndex`` 获得。很不爽，实际使用场景里面大家直接把 ``String`` 当 Unicode 字符串搞。假定无风险好了。

用 ``sizeofValue`` 检查 ``aString.endIndex`` 发现大小是 32，猜测是 4x8 (64位)个整数或者指针，如果是我，索引值肯定放在这个结构的第一位，所以这就是 ``_Dummy`` 结构的由来（猜测+代码验证），就是为了和 ``String.Index`` 类型大小一致结构相似（反正我只关心第一个，其他的我用 ``_padding`` 补全）

然后用强制类型转换，然后操作这个值。

至于重载 subscript ，那啥。看书好了。

然后有人说字符串没有长度方法。。

``countElements(aString)`` 标准库函数。可以用 extension 做到 ``String`` 里。

```scala
extension String {
    struct _Dummy {
        var idxVal: Int
        var _padding: Int
        var _padding2: Int
        var _padding3: Int
    }
    subscript (i: Int) -> Character {
        var dummy: _Dummy = reinterpretCast(i >= 0 ? self.startIndex : self.endIndex)
        dummy.idxVal += i
        let idx: String.Index = reinterpretCast(dummy)
        return self[idx]
    }
    subscript (subRange: Range<Int>) -> String {
        var start: _Dummy = reinterpretCast(self.startIndex)
        var end = start
        start.idxVal = subRange._startIndex
        end.idxVal = subRange._endIndex
        let startIndex: String.Index = reinterpretCast(start)
        let endIndex: String.Index = reinterpretCast(end)
        return self[startIndex..endIndex]
    }
}
```

讨论内容在 [SwiftChina](http://swift.sh/topic/95/stringindexrange/). 其中有人给出了负数 range 的实现。
