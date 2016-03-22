---
layout: post
title: "Swift beta3 Changes ( Swift 在 beta3 中的变化）"
date: 2014-07-08 17:22:11 +0800
comments: true
categories: swift swift-internals
---

准确说是 beta2 ``Swift version 1.0 (swift-600.0.34.4.8)`` 到 beta3 ``Swift version 1.0 (swift-600.0.38.7)`` 的变化。

对了，补充下。 beta1 ``Swift version 1.0 (swift-600.0.34.4.5)`` 到 beta2 几乎没有什么变化。

## 语法

``nil`` 成为关键字。

``[KeyType : ValueType]`` 可以表示字典类型 ``Dictionary<KeyType, ValueType>``。

``[Type]`` 用于表示原 Array 类型 ``Type[]``，等价 ``Array<T>``，原用法会导致警告。

~~增加 @noinline 属性~~

``..`` 运算符改为 ``..<``，不容易和 ``...`` 混淆。

## 函数、类型

原 ``sort()`` 改名为 ``sorted()``。新增 ``sort()`` 函数，参数为 ``inout``。

Index 类型中的 ``.succ()`` 变为 ``.successor()``、 ``.pred()`` 变为 ``.predecessor()``。

## C/ObjC 交互变化

增加 ``UnsafeMutableArray<T>`` 类型。

增加 ``CFunctionPointer<T>`` 类型。

删除 ``CConstVoidPointer``、 ``CMutableVoidPointer``。替换为 ``UnsafePointer<()>``、``ConstUnsafePointer<Int32>``。

删除 ``CConstPointer<T>``、``CMutablePointer<T>``。替换为 ``UnsafePointer<T>``、``ConstUnsafePointer<T>``。

这么一来指针操作简单了好多。原有会出现 ``COpaquePointer`` 的不合理情况，也都对应到适合的类型。

``CString`` 可以从 ``UnsafePointer<UInt8>`` 和 ``UnsafePointer<CChar>`` 两种类型构造获得，之前只支持 ``UInt8``。

module.map 中头文件声明转换为 Swift 声明不再使用 C 兼容类型，直接使用 Swift 相应类型。原有 ``CInt``，现在成为 ``Int32``。

结构体会自动添加构造函数 ``init(field1:field2:...)`` 这样。

### nil

去掉了 ``NilType``，增加了 ``NilLiteralConvertible``， ``nil`` 成为关键字。可以认为是 nil 常量。

```
protocol NilLiteralConvertible {
  class func convertFromNilLiteral() -> Self
}
```

除了 Optional 、上面所提到的指针类型外，``RawOptionSet`` 也实现了该协议。

### Array

去掉了 ``.copy()``、``unshare()`` 方法。

增加了以下方法：

```
func makeUnique(inout buffer: ArrayBuffer<T>, e: T, index: Int)
func sorted(isOrderedBefore: (T, T) -> Bool) -> Array<T>
```

看起来 ``Array`` 对底层容器的引用有了更好的控制 ``ArrayBufferType`` 增加了判断方法 ``func isMutableAndUniquelyReferenced() -> Bool``。

Array 目前可以认为是真正的值类型。

### 指针

#### 增加了 ``_Pointer`` protocol

```scala
protocol _Pointer {
  var value: RawPointer { get }
  init(_ value: RawPointer)
}
```

表示一个类型可以对应到原生指针。

同时成为内部桥接类型，编译器内部在转换时使用它（取出 RawPointer, 构造具体指针类型）。

## 模块

增加了  StdlibUnittest 模块。 [声明代码](https://github.com/andelf/Defines-Swift/blob/79ed8d40659e4d038f41e3c30b4b3358106bd50a/StdlibUnittest.swift)。单元测试终于有了。

