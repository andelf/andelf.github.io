---
layout: post
title: "Swift and ObjectiveC Interop (Swift 与 Objective-C 之间的交互)"
date: 2014-06-11 20:01:11 +0800
comments: true
categories: swift swift-interop
---

主要介绍原有 Objective-C 中的一些类型在 Swift 中的操作问题。差不多就能知道，你在 Swift 如何对 Objective-C (Foundation) 原生类型进行操作以及和 Swift 类型互转。

瞎写的，有错或者不明白的地方直接告诉我。转载注明我或者 SwiftChina 。

参考资料是 我的 Github : [andelf/Defines-Swift](https://github.com/andelf/Defines-Swift) 是我dump出的定义。10w行swift代码。

# 预备知识

## Objective-C (Cocoa) 特色

数值类型统一用 ``NSNumber``，不区分具体。大量操作在指针上进行，也就隐含着一切可能出错的调用都会返回 ``nil``。
没有泛型支持，字典或者数组更多用 ``id`` 类型。

## 对应 Swift 特色

数值类型众多 （``Int, UInt, Float, Double, ...``），无指针（``UnsafePointer`` 这类用于和其他代码交互的结构不计，``&`` + ``inout`` 认为是引用。）。
``nil`` 用于 ``Type?``, ``Type!``。字典和数组用泛型表示。

## 一些属性

有几个属性用于和 Objective-C 之间的交互。

### ``@objc`` 和 ``@objc(a_name_defined_in_objc)``

### ``@objc_block``

用于函数类型前，标记后面的 block 是 Objective-C 的。个人觉得编译器会自动转换相关的 block 类型。

然后需要预备下几个属性的知识。

### ``@conversion``

Swift 中的隐式类型转换。有另一篇文章介绍。

### ``@final``

就是 final 了。

### ``@transparent``

经过反汇编查看，相当于 ``inline`` 的作用。同时生成的 ``.swiftmodule`` 文件依然带有原函数实现。这个很高大上。保证了 ``inline`` 特性在包外可用。

## 重要 Swift protocol

### ``*LiteralConvertible``

表示可以从 Swift 字面常量转换而得。比如 ``let a: Type = 100``, 那么 Type 必须实现 IntegerLiteralConvertible。

大概有 IntegerLiteralConvertible, FloatLiteralConvertible, StringLiteralConvertible, ArrayLiteralConvertible, DictionaryLiteralConvertible, CharacterLiteralConvertible 等。

### ``_BridgedToObjectiveC`` ``_ConditionallyBridgedToObjectiveC``

顾名思义。就是相当于这个 Swift 类型可以和某一 Objective-C 类型对等，可以相互转换。

Conditional 多了个判断函数，也就是说这个类型可能是并没有对等起来的，什么情况下用呢，目前猜测应该是带泛型参数的类型中用到，有的泛型可以对应，有的不可以。

```scala
protocol _BridgedToObjectiveC {
  typealias ObjectiveCType
  class func getObjectiveCType() -> Any.Type
  func bridgeToObjectiveC() -> ObjectiveCType
  class func bridgeFromObjectiveC(source: ObjectiveCType) -> Self?
}
```

### ``Reflectable`` ``Mirror``

这个具体暂时不清楚。反射相关的两个 protocol 。

### ``Sequence`` ``Generator``

这两个用于 Swift 的 for-in 循环，简单说就是，实现了 ``Sequence`` 协议的对象可以 ``.generate()`` 出一个 ``Generator``，
然后 ``Generator`` 可以不断地 ``.next()`` 返回 ``Type?``，其中 ``Type`` 是这个序列的泛型。

### ```Hashable``` ```Equatable```

英汉字典拿来。所以其实为了让 Objective-C 类型在 Swift 代码中正常工作，这个是必不可少的。

# 第一部分，类型交互

## For 类、方法、协议

通过 ``@objc(name)`` 转为 Swift 定义。其中 name 为 Objective-C 下的名字。

例如

```
@objc(NSNumber) class NSNumber : NSValue {
    @objc(init) convenience init()
    @objc var integerValue: Int {
        @objc(integerValue) get {}
    }
    ...
}
@objc(NSCopying) protocol NSCopying {
    @objc(copyWithZone:) func copyWithZone(zone: NSZone) -> AnyObject!
}
```

所有可能为 ``nil`` 的指针类型几乎都被转为 ``Type!``，由于 ``ImplicitlyUnwrappedOptional`` 的特性，所以几乎用起来一样。

## For 基础数字类型

``Int``, ``UInt``, ``Float``, ``Double`` 均实现了 ``_BridgedToObjectiveC``, 其中类型参数 ``ObjectiveCType`` 均为 ``NSNumber``。

也就是说，可以直接 ``.getObjectiveCType()`` 获取到 ``NSNumber``，然后剩下的就很熟悉了。

``NSNumber`` 实现了 ``FloatLiteralConvertible``, ``IntegerLiteralConvertible``，所以其实，也可以直接从 Swift 字面常量获得。

## For Bool

实际上 Objective-C 中的 ``BOOL`` 是某一数字类型，``YES``, ``NO`` 也分别是 1, 0 。

所以 Swift ``Bool`` 实现了 ``_BridgedToObjectiveC``，对应于 ``NSNumber`` 类型。

## For String

``NSString`` 实现了 ``StringLiteralConvertible``，可以直接通过字面常量获得，同时还有到 ``String`` 的隐式类型转换。

``String`` 实现了 ``_BridgedToObjectiveC`` 对应于 ``NSString``。

```scala
extension NSString {
  @conversion func __conversion() -> String
}
```

这就是为什么官方文档说 Swift 中 ``String`` 和 Objective-C 基本是相同操作的。其实都是背后的隐式类型转换。

同时 Foundation 还为 ``String`` 扩充了很多方法，发现几个比较有意思的是 ``._ns``，直接返回 ``NSString``, ``._index(Int)`` 返回 ``String.Index``，等等。

## For Array

``NSArray`` 实现了 ``ArrayLiteralConvertible``， 可以从字面常量直接获得。还实现了到 ``AnyObject[]`` 的隐式类型转换。

``Array<T>`` 实现了 ``_ConditionallyBridgedToObjectiveC``, 对应于 ``NSArray``. 

``NSArray`` 还实现了 ``Sequence`` 协议，也就是可以通过 for-in 操作。其中 ``generate()`` 返回 ``NSFastGenerator`` 类，这个应该是在原有 Foundation 没有的。
当然 ``.next()`` 返回 ``AnyObject?``

## For Dictionary

``NSDictionary`` 实现了 ``DictionaryLiteralConvertible``, ``Sequence``。同时还实现了到 ``Dictionary<NSObject, AnyObject>`` 的隐式类型转换。

``Dictionary<KeyType, ValueType>`` 实现了到 ``NSDictionary`` 的隐式类型转换。实现了 ``_ConditionallyBridgedToObjectiveC`` 对应于 ``NSDictionary``。

## 其他扩充类型

新类型 ``NSRange``，实现了 ``_BridgedToObjectiveC``，对应于 ``NSValue``，实现了到 ``Range<Int>`` 的隐式类型转换。

``NSMutableSet`` ``NSMutableDictionary`` ``NSSet`` ``NSMutableArray`` 均实现了 ``Sequence`` 可以 for-in .

# 第二部分：从 Swift 调用 Objective-C

一般说来，你在 Swift 项目新建 Objective-C 类的时候，直接弹出是否创建 Bridge Header 的窗口，点 YES 就是了，这时候一般多出来个 ``ProjectName-Bridging-Header.h`` 。

如果没有自动的话，这个配置在项目的 Build Settings 中的 Swift Compiler - Code Generation 子项里。

说到底，其实是调用编译命令的 ``-import-objc-header`` 参数，后面加上这个 Header 文件。

然后你就可以把你的 Objective-C Class 的 .h 文件都 import 到这个 Herder 文件里了。

所有 Swift 代码都可以直接调用。完全透明，自动生成。

# 第三部分：从 Objective-C 调用 Swift

头文件是 ``ProjectName-Swift.h``。直接 import 。

不要吝啬你的键盘大量地加入 ``@objc`` 属性就是了。

具体实现据我猜测是这样，先 swift 调用 ``-emit-objc-header`` ``-emit-objc-header-path`` 参数控制生成 Objective-C 的 Header，同时 swift 编译为模块，然后再编译一次。

头文件内容大概是会包含一堆宏定义。然后是 Swift 的类定义等。这里可以看到 Swift 的 mangling 名字。

```scala
SWIFT_CLASS("_TtC5Hello11AppDelegate")
@interface AppDelegate : UIResponder <UIApplicationDelegate>
@property (nonatomic) UIWindow * window;
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
- (void)applicationWillResignActive:(UIApplication *)application;
- (void)applicationDidEnterBackground:(UIApplication *)application;
- (void)applicationWillEnterForeground:(UIApplication *)application;
- (void)applicationDidBecomeActive:(UIApplication *)application;
- (void)applicationWillTerminate:(UIApplication *)application;
- (instancetype)init OBJC_DESIGNATED_INITIALIZER;
@end
```

# 第四部分：一个 Swift 和 Objective-C 混合项目的编译过程

这里只先考虑一个 Swift 项目使用 Objective-C 代码的情况，这个应该暂时比较多见（使用旧的 MVC 代码，用新的 Swift 创建 ui 一类）。

- 编译所有 ``X.swift`` 文件到 ``X.o`` (with ``-emit-objc-header``, ``-import-objc-header``) (其中包含 ``.swiftmodule`` 子过程)
  - 由于选项里有 ``-emit-objc-header``，所以之后的 Objective-C 文件可以直接 import 对应的 ``ProjectName-Swift.h``
- 编译 ``X.m`` 到 ``X.o``
- 链接所有 ``.o`` 生成可执行文件


