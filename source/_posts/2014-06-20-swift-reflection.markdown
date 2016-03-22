---
layout: post
title: "Swift Reflection （Swift 的反射）"
date: 2014-06-20 23:48:51 +0800
comments: true
categories: swift
---

Swift 其实是支持反射的，不过功能略弱。本文介绍基本的反射用法和相关类型。

## MetaType 和 Type 语法

> The metatype of a class, structure, or enumeration type is the name of that type followed by .Type. The metatype of a protocol type—not the concrete type that conforms to the protocol at runtime—is the name of that protocol followed by .Protocol. For example, the metatype of the class type SomeClass is SomeClass.Type and the metatype of the protocol SomeProtocol is SomeProtocol.Protocol.

> You can use the postfix self expression to access a type as a value. For example, SomeClass.self returns SomeClass itself, not an instance of SomeClass. And SomeProtocol.self returns SomeProtocol itself, not an instance of a type that conforms to SomeProtocol at runtime.

- metatype-type -> ``type``.**Type** | ``type``.**Protocol**
- type-as-value -> ``type``.self

其中 metatype-type 出现在代码中需要类型的地方， type-as-value 出现在代码中需要值、变量的地方。

``Any.Type`` 类型大家可以猜下它表示什么。

## 基础定义

反射信息用 ``Mirror`` 类型表示，类型协议是 ``Reflectable``，但实际看起来 ``Reflectable`` 没有任何作用。

```
protocol Reflectable {
  func getMirror() -> Mirror
}
protocol Mirror {
  var value: Any { get }
  var valueType: Any.Type { get }
  var objectIdentifier: ObjectIdentifier? { get }
  var count: Int { get }
  subscript (i: Int) -> (String, Mirror) { get }
  var summary: String { get }
  var quickLookObject: QuickLookObject? { get }
  var disposition: MirrorDisposition { get }
}
```

实际上所有类型都实现了 ``Reflectable``。

``Mirror`` 协议相关字段：

- ``value`` 相当于变量的 ``as Any`` 操作
- ``valueType`` 获得变量类型
- ``objectIdentifier`` 相当于一个 UInt 作用未知，可能是 metadata 表用到
- ``count`` 子项目个数（可以是类、结构体的成员变量，也可以是字典，数组的数据）
- ``subscript(Int)`` 访问子项目, 和子项目的名字
- ``summary`` 相当于 ``description``
- ``quickLookObject`` 是一个枚举，这个在 WWDC 有讲到，就是 Playground 代码右边栏的显示内容，比如常见类型，颜色，视图都可以
- ``disposition`` 表示变量类型的性质，基础类型 or 结构 or 类 or 枚举 or 索引对象 or ... 如下



```
enum MirrorDisposition {
  case Struct // 结构体
  case Class // 类
  case Enum // 枚举
  case Tuple // 元组
  case Aggregate // 基础类型
  case IndexContainer // 索引对象
  case KeyContainer // 键-值对象
  case MembershipContainer // 未知
  case Container // 未知
  case Optional // Type?
  var hashValue: Int { get }
}
```

通过函数 ``func reflect<T>(x: T) -> Mirror`` 可以获得反射对象 ``Mirror`` 。它定义在 ``Any`` 上，所有类型均可用。

## 实际操作

### ``.valueType`` 处理

``Any.Type`` 是所有类型的元类型，所以 ``.valueType`` 属性表示类型。实际使用的时候还真是有点诡异：

```
let mir = reflect(someVal)
swith mir.valueType {
case _ as String.Type:
    println("type = string")
case _ as Range<Int>.Type:
    println("type = range of int")
case _ as Dictionary<Int, Int>.Type:
    println("type = dict of int")
case _ as Point.Type:
    println("type = a point struct")
default:
    println("unkown type")
}    
```

或者使用 ``is`` 判断：

```
if mir.valueType is String.Type {
    println("!!!type => String")
}
```

勘误： 这里之前笔误了。遗漏了 ``.valueType`` 。

``is String`` 判断变量是否是 String 类型，而 ``is String.Type`` 这里用来判断类型是否是 String 类型。

### ``subscript(Int)`` 处理

实测发现直接用 ``mir[0]`` 访问偶尔会出错，也许是 beta 的原因。

```
for r in 0..mir.count {
    let (name, subref) = mir[r]
    prtln("name: \(name)")
    // visit sub Mirror here
}
```

通过上面的方法，基本上可以遍历大部分结构。

## 不同类型的处理

### Struct 结构体、 Class 类

- ``.count`` 为字段个数。
- ``subscript(Int)`` 返回 （字段名，字段值反射 ``Mirror``） 元组
- ``summary`` 为 mangled name

### Tuple 元组

- ``.count`` 为元组子元素个数
- ``subscript(Int)`` 的 name 为 ".0", ".1" ...

### Aggregate 基础类型

包括数字、字符串（含 ``NSString``）、函数、部分 Foundation 类型、 MetaType 。

很奇怪一点是测试发现枚举也被反射为基础类型。怀疑是没实现完。

- ``.count`` 为 0

### IndexContainer 索引对象

包括 ``Array<T>``, ``T[]``, ``NSArray`` 等。可以通过 subscript 访问。

- ``.count`` 为元组子元素个数
- ``subscript(Int)`` 的 name 为 "[0]", "[1]" ...

### KeyContainer 键-值对象

包括 ``Dictionary<T, U>``、``NSDictionary``

- ``.count`` 为元组子元素个数
- ``subscript(Int)`` 的 name 为 "[0]", "[1]" ... 实际访问是 ``(name, (reflect(key), reflect(val)))``

### Optional Type?

只包括 ``Type?``，不包括 ``Type!``。

- ``.count`` 为 0 或者 1 (对应 nil 和有值的情况)
- ``subscript(Int)`` , name 为 "Some"

### 其他

- Enum 枚举 看起来是未使用
- MembershipContainer // 未知
- Container // 未知


## 示例代码

[Gist](https://gist.github.com/anonymous/190fc7ecee30e83a5dca)


