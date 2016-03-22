---
layout: post
title: "Swift Type Hierarchy ( Swift 类型层次结构 ）"
date: 2014-06-30 15:10:41 +0800
comments: true
categories: swift swift-internals
---

声明： 转载请注明，方便的情况下请知会本人. [weibo](http://weibo.com/234632333)

本文主要介绍 Swift 所有标准库类型的层次结构，及所有标准类型。本文可作为参考手册使用。

本人不保证内容及时性和正确性，请善于怀疑并反馈。谢谢。

本文探索 Swift 所有基础类型和高级类型，以及所有协议和他们之间的继承关系。

为了简化问题，某些类型略去了中间的过渡类型，人肉保证不歧义。

## Swift 基础类型

### 数值类型

#### 位

    Bit

只有一位，实现为 ``enum``， ``.zero`` 或 ``.one``。简单明了。

协议: ``RandomAccessIndex IntegerArithmetic``

#### 整型

有符号:
```
Int Int8 Int16 Int32 Int64
```

协议：``SignedInteger RandomAccessIndex BitwiseOperations SignedNumber CVarArg``

无符号:
```
UInt UInt8 UInt16 UInt32 UInt64
```
协议：``UnsignedInteger RandomAccessIndex BitwiseOperations``

别名:

```
IntMax = Int64
UIntMax = UInt64
IntegerLiteralType = Int
Word = Int // 字长
UWord = UInt
```

#### 浮点型
```
Float Double Float80
```

别名：

```
FloatLiteralType = Double
Float64 = Double
```

协议：``FloatingPointNumber``。

### 逻辑型

只有一个 ``Bool``。

实例： ``true``、``false``

协议：``LogicValue``。

### 空

只有一个 ``NilType``。
    
唯一实例 ``nil``。

### 字符（串）类型

- ``String``
- ``Character`` Unicode 字符
- ``UnicodeScalar`` 相当于 C 中的 ``wchar_t``
- ``CString`` 用于表示 C 中的 ``const char *``，请参考相关文章
- ``StaticString`` 静态字符串，内部使用，例如 ``fatalError``

别名：

```
StringLiteralType = String
ExtendedGraphemeClusterType = String
```

官方文档

> `Character` represents some Unicode grapheme cluster as
> defined by a canonical, localized, or otherwise tailored
> segmentation algorithm.

``String`` 实现协议：``Collection ExtensibleCollection OutputStream TargetStream``。

### Array 类型

- ``Array<T>``
- ``ContiguousArray<T>``

实现协议 ``ArrayType``。

内部容器：

- ``ArrayBuffer<T>``
- ``ContiguousArrayBuffer<T>``

这两个类型看起来是 Array 的内部容器，一般不应该直接使用。

### 字典类型

``Dictionary<KeyType : Hashable, ValueType>``

只实现了 ``Collection``。

### 元祖类型

除正常元祖外，还有个特殊的别名

    Void = ()

其实很多语言都这么定义的，比如 Haskell 。

### Optional 类型

- ``Optional<T>`` 即 ``T?``
- ``ImplicitlyUnwrappedOptional<T>`` 即 ``T!``

实现协议: ``LogicValue``，行为是判断是否为 ``.None``。

另外 [Swift 的隐式类型转换](http://andelf.github.io/blog/2014/06/08/swift-implicit-type-cast/)
有提到，为什么 ``nil`` 可以给 ``Optional`` 类型赋值的问题。

### C/ObjC 兼容类型

```
CBool = Bool
CFloat = Float
CDouble = Double
CChar = Int8
CSignedChar = Int8
CUnsignedChar = UInt8
CChar16 = UInt16
CWideChar = UnicodeScalar
CChar32 = UnicodeScalar
CInt = Int32
CUnsignedInt = UInt32
CShort = Int16
CUnsignedShort = UInt16
CLong = Int
CUnsignedLong = UInt
CLongLong = Int64
CUnsignedLongLong = UInt64
```

具体使用参考 C 交互的几篇文章，基本没区别。

### Any 类型

```
AnyObject
// 别名
Any = protocol<>
AnyClass = AnyObject.Type
```

还有个用在函数定义的类型签名上， ``Any.Type``。

顺便这里看到一个奇异的语法 ``protocol<>``，这个也是 Swift 一种用来表示类型限制的方法，可以用在类型的位置，尖括号里可以是协议的列表。

### 指针类型

```
UnsafePointer<T>
CMutableVoidPointer
CConstVoidPointer
COpaquePointer
CConstPointer<T>
AutoreleasingUnsafePointer<T>
CVaListPointer
CMutablePointer<T>
```

参考 C 交互文章。

- [简析Swift和C的交互](http://andelf.github.io/blog/2014/06/15/swift-and-c-interop/)
- [简析 Swift 和 C 的交互，Part 二](http://andelf.github.io/blog/2014/06/18/swift-and-c-interop-cont/)
- [Swift 与 ObjC 和 C 的交互，第三部分](http://andelf.github.io/blog/2014/06/28/swift-interop-with-c-slash-objc/)

### 其他辅助类型

多了去了。比如 for-in 实现时候的 Generator 、比如反射时候用的 ``*Mirror``、比如切片操作用的 ``Range<T>``。比如内部储存类。

还有储存辅助类 ``OnHeap<T>`` 等等。以后有机会再探索。

## Swift 标准库协议

### 打印相关 ``Printable DebugPrintable`` 

```scala
protocol Printable {
  var description: String { get }
}
protocol DebugPrintable {
  var debugDescription: String { get }
}
```

用于打印和字符串的 Interpolation 。

### ``*LiteralConvertible``

从字面常量获取。

```
ArrayLiteralConvertible
IntegerLiteralConvertible
DictionaryLiteralConvertible
CharacterLiteralConvertible
FloatLiteralConvertible
ExtendedGraphemeClusterLiteralConvertible
StringLiteralConvertible
```

其中字符串和字符的字面常量表示有所重合，也就是说 ``"a"`` 可以是字符串也可以是字符。[简析 Swift 中的 Pattern Match](http://andelf.github.io/blog/2014/06/17/nsobject-pattern-match-in-swift/) 一文中就是遇到了类似的情况。

### ``LogicValue``

相当于重载 ``if``、``while`` 的行为。

```scala
protocol LogicValue {
  func getLogicValue() -> Bool
}
```

### ``Sequence``

相当于重载 for-in 。和 ``Generator`` 联用。

```scala
protocol Sequence {
  typealias GeneratorType : Generator
  func generate() -> GeneratorType
}

protocol Generator {
  typealias Element
  mutating func next() -> Element?
}
```

```scala
// for .. in { }
var __g = someSequence.generate()
while let x = __g.next() {
    ...
}}
```

### 整型、 Index 相关协议

这些协议都是用来表示容器类型的索引、及相关的索引运算。

这里略去了部分私有内容。略去了 ``Printable`` 等。

{% img left /images/attachments/swift-integer.png 'Swift Integer Type Hierarchy' %}

### RawOptionSet 相关协议

一般用来表示二进制的选项，类似于 C enum ，很多 Cocoa 的 flag 被映射到它。相当于一个 Wrapper 的作用。

{% img left /images/attachments/swift-rawoptset.png 'Swift RawOptionSet' %}

可以看到要求被 Wrap 的对象支持 ``BitwiseOperations``。

### Array 相关协议

图中用虚线标注了和 ``Generator`` 的关系。

{% img left /images/attachments/swift-collection.png 'Swift Collection Protocol' %}

``Array<T>`` 类型实现了 ``ArrayType`` 协议。

``Dictionary`` 类型实现了 ``Collection`` 协议。

### 反射相关协议

包括 ``Mirror``、``MirrorDisposition``、``Reflectable``。

请参考 [Swift 的反射](http://andelf.github.io/blog/2014/06/20/swift-reflection/)。

### 浮点数协议

只有一个 ``FloatingPointNumber``。单独存在。是为了定义完整而存在。看官自己搞定。

### IO 输出，伪输出相关

```
protocol Streamable {
  func writeTo<Target : OutputStream>(inout target: Target)
}
```

``Streamable`` 表示可以被写入到输出流中，比如字符串、字符等。

```
protocol OutputStream {
  func write(string: String)
}
```

``OutputStream`` 表示一个输出流，比如标准输出（``stdout``），也可以表示一个伪输出流，例如字符串 ``String``。

标准输出的获取方法

    var stdout = _Stdout()
    
看起来是私有结构，某一天不能用的话，别怪我。调用时候用 ``inout`` 引用语法。

### CVarArg 处理

```
protocol CVarArg {
  func encode() -> Word[]
}
```

用于处理 C 函数的可变参数，参考 [简析 Swift 和 C 的交互，Part 二](http://andelf.github.io/blog/2014/06/18/swift-and-c-interop-cont/)。

### Bridge 协议

这里有个疑问就是编译过程中这些 Bridge 协议有没有参与。目前还没办法确定。

- ``_BridgedToObjectiveC``
- ``_ConditionallyBridgedToObjectiveC``

具体内容可以参考 [Swift 与 Objective-C 之间的交互](http://andelf.github.io/blog/2014/06/11/swift-and-objectivec-interop/)一文。

### 其他

``Sink`` 看起来是一个容器，可能是用来编码时使用。

``ArrayBufferType`` 用于表示 ``ArrayType`` 的内部储存，看起来似乎也可以直接用。

``UnicodeCodec`` 用于处理编码。有 ``UTF8``、``UTF16``、``UTF32`` 可用。

``ArrayBound`` 用来处理数组边界，详细原理和作用过程未知。

## 总结

无。

参考：

- [我的 Github andelf/Defines-Swift](https://github.com/andelf/Defines-Swift)
