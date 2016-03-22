---
layout: post
title: "Swift and C Interop(简析Swift和C的交互)"
date: 2014-06-15 00:11:44 +0800
comments: true
categories: swift swift-interop
---

声明： 转载注明我或者 SwiftChina . [weibo](http://weibo.com/234632333)

其中 ``@asmname`` 的两个用法源于我的猜测验证，用到了 Xcode, lldb, nm, llvm ir 等工具或格式。

其中 name mangling 部分源自 WWDC。

相关的分析主要基于我 dump 出的 Swift 标准库声明代码，位于 [我的 Github andelf/Defines-Swift](https://github.com/andelf/Defines-Swift)。

目前有两个小部分还未完成。

之前好像简单说过 Swift 和 Objective-C 的交互问题。其实我们也可以用 Swift 调用纯 C 代码或者基于 C 的第三方库。（这里不会也永远不会考虑 C++ 的情况，因为不支持，不过可以用 C 写 wrapper, 这个没有任何问题。）

Swift 官方文档中，以及那本已经被迅速翻译为中文的 ibooks 书中，都提到了 Swift 调用 Objective-C 和 C 是有很好支持的。不过没有细节。

~~这里不会考虑如何用 C 调用 Swift, 暂时，看心情。：）~~ 本内容包括 Swift 调用 C 和相应的 C 调用 Swift，项目混编。

这里主要面向 MacOSX 应用。iOS 或许可以适用。

先复习下区别。

## 第一部分 预备知识

### 语言区别

说到底就是 C 少了很多东西。但是多了个指针。

对于 C 来说，最头疼的莫过于指针，而 Swift 是一门没有指针的语言。是不是要吐血？相对来说指针不但代表者指针操作传参，还有指针运算等等。

## 第二部分 调用 C

这里主要讨论函数的调用。对于结构、枚举、类的兼容性暂时没尝试。

### C 标准库

好消息是，对于标准库中的 C 函数，根本不需要考虑太多导入头文件神马的。比如 ``strlen``、``putchar``、``vprintf``。当然 ``vprintf`` 需要多说几句，后面说。

请直接 ``import Darwin`` 模块。

这些标准库函数表示为 ``Darwin.C.HEADER.name``。

实际上由于 Swift 模块结构是平坦的，他们均位于 ``Darwin`` 中，所以基本上是直接用的。

然后 ``CoreFoundation`` 用到了 ``Darwin`` ( @exported 导入，所以相当于这些名字也在 ``CoreFoundation`` 中)。

然后 ``Foundation`` 用到了 ``CoreFoundation`` （也是 @exported 导入。）

所以其实你导入 ``Foundation`` 的时候，这些 C 函数都是直接可用的。比如 ``putchar`` 一类。

多说一句，``Cocoa`` 当然也包含 ``Foundation``。

### C 函数

好吧假设你有个牛逼到顶天的算法是 C 写的，不对，假设是别人写的，牛逼到你只能凑合用却看不懂然后自己也写不出没空迁移的地步。

我们直接创建一个 Swift 项目，然后 New File，添加一个 ``.c`` 文件。

这时候 Xcode 会弹出对话框询问是否配置 Bridge Header，确认就可以了。也可以手动添加 Bridge Header，位置在项目的 Build Settings 中的 Swift Compiler - Code Generation 子项里。指向你的 Bridge Header 文件名就可以了。

一般这个文件是 ``ProjectName-Bridging-Header.h``。情况基本和与 Objective-C 混编没区别。

剩下的工作就很简单了。在 ``.c`` 文件填上传说中的牛逼算法。在 ``ProjectName-Bridging-Header.h`` 中加上该函数原型或者引入相关的头文件。

在 Swift 中调用的名字和 C 名字一样就可以了，比如你定义了一个 ``int mycsort()`` 那么在 Swift 中就是 ``func mycsort() -> CInt``。

这时候问题来了。一个漂亮的问题。

我的 C 函数名字和 Swift 标准库冲突怎么办？比如我定义了一个函数就叫 ``println``，我们知道 Swift 里也有个 ``println``。

这样，如果直接调用会提示 ``Ambiguous use of 'println'``。没辙了么？

这里有个我发现的 Undocumented Featuer 或者说是 Undocumented Attribute。你转载好歹提下我好吧。（发现方法是通过 Xcode 查看定义，然后通过 nm 命令发现符号, 对照 llvm ir 确认的。)

那就是 ``@asmname("func_name_in_c")``。用于函数声明前。使用方法：

```c
int println() { .... }
```

```
@asmname("println") func c_println() -> CInt // 声明，不需要 {} 函数体
c_println() // 调用
```

也就是 C 中的同名函数，我们可以给赋予一个别名，然后正常调用。这么一看就基本没有问题了。至于类型问题，待会说，详细说。

### C Framework

很多时候我们拿到的是第三方库，格式大概是个 Framework。比如 ``SDL2.framework``。举这个例子是因为我想对来说比较熟悉 SDL2。

直接用 Finder 找到这个 ``.framework`` 文件，拖动到当前项目的文件列表里，这样它将作为一个可以展开的文件夹样式存在于我们的项目中。

在 ``ProjectName-Bridging-Header.h`` 中引入其中需要的 ``.h``。

比如我们引入 ``SDL2.framework``，那么我们就需要写上 ``#import <SDL2/SDL.h>``。

然后在 Swift 文件里正常调用就好了。

所以其实说到底核心就是那个 ``ProjectName-Bridging-Header.h``，因为它是作为参数传递给 Swift 编译器的，所以 Swit 文件里可以从它找到定义的符号。

但是，这个桥接头文件的一切都是隐式的，类型自动对应，所以很多时候需要我们在 Swift 里调用并封装。或者使用 ``@asmname(...)`` 避免名字冲突。

## 第三部分 类型转换

前面说到了 C 中有指针，而 Swift 中没有，同时基本类型还有很多不同。所以混编难免需要在两种语言的不同类型之间进行转换。

牢记一个万能函数 ``reinterpretCast<T, U>(T) -> U``，只要 T, U sizeof 运算相等就可以直接转换。这个在之前的标准库函数里有提到。调用 C 代码的利器！

### 基本类型对应

**以下内容再 Xcode6-beta3 中不适用** 请参考 [Swift 在 Xcode6-beta3 中的变化](http://andelf.github.io/blog/2014/07/08/swift-beta3-changes/)。简言之，不在对应为 C 别名类型，而是直接对应到 Siwft 类型。而指针类型简化为 ``UnsafePointer<T>`` 和 ``ConstUnsafePointer<T>`` 两种， ``COpaquePointer`` 依然存在。另新增了 ``CFunctionPointer<T>``。

- int => CInt
- char => CChar / CSignedChar
- char* => CString
- unsigned long = > CUnsignedLong
- wchar_t => CWideChar
- double => CDouble
- T* => CMutablePointer<T>
- void* => CMutableVoidPointer
- const T* => CConstPointer<T>
- const void* => CConstVoidPointer
- ...

继续这个列表，你肯定会想这么多数值类型，怎么搞。其实大都是被 ``typealias`` 定义到 ``UInt8``，``Double`` 这些的。放心。C 中数值类型全部被明确地用别名定义到带 size 的 Swift 数值类型上。完全是一样用的。

其实真正的 Pointer 类型只是 ``UnsafePointer<T>``，大小与 C 保证一致，而对于这里不同类型的 Pointer，其实都是 ``UnsafePointer``
到它们的隐式类型转换。~~还有个指针相关类型是 ``COpaquePointer``，不过没试验怎么用~~。

UPDATE: 我们在调用的时候，更多地用到 ``COpaquePointer``，我将再坑一篇介绍它。

同时 ``NilType``，也就是 ``nil`` 有到这些指针的隐式类型转换。所以可以当做任何一种指针的 ``NULL`` 用。

还有个需要提到的类型是 CString, 他的内存 layout 等于 ``UnsafePointer<UInt8>``，下面说。

### CString 

用于表示 ``char *``，``\0`` 结尾的 c 字符串，实际上似乎还看到了判断是否 ASCII 的选项，但是没试出来用法。

实现了 ``StringLiteralConvertible`` 和 ``LogicValue``。可以从字符串常量直接赋值获得 ``CString``。``LogicValue`` 也就是是 ``if a_c_str {}``，实际是用于判断是否为 ``NULL``，可用，但是不稳定，老 crash。

运算符支持 ``==``，判断两字符串是否相当，猜测实际是 ``strcmp`` 实现，对比 NULL 会 crash。Orz。

CString 和 String 的转换通过一个 extension 实现，也是很方便。

```scala
extension String {
  static func fromCString(cs: CString) -> String
  static func fromCString(up: UnsafePointer<CChar>) -> String
}
// 还有两个方便的工具方法。 Rust 背景的同学一定仰天长啸。太相似了。
extension String {
  func withCString<Result>(f: (CString) -> Result) -> Result
  func withCString<Result>(f: (UnsafePointer<CChar>) -> Result) -> Result
}
```

在我们的 Bridging Header 头文件中 ``char *`` 的类型会对应为 ``UnsafePointer<CChar>``，而实际上 ``CString`` 更适合。所以在 Swift 代码中，往往我们要再次申明下这个函数。或者用一个函数封装下，转换成我们需要的类型。

例如，假设在 Bridging Header 中我们声明了 ``char * foo();``，那么，在 Swift 代码中我们可以用上面提到的方法：

```
@asmname("foo") func c_foo() -> CString
// 注意这里没有 {}，只是声明
let ret = c_foo()
```

当然也可以直接调用原始函数然后类型转换：

```scala
let raw = foo() // UnsafePointer<Int8> <=> UnsafePointer<CChar>
let ret = String.fromCString(ret)
```

如果这里要转成 CString 就略复杂了，因为 CString 构造函数接受的参数是 ``UnsafePointer<UInt8>``， 而 ``CChar`` 是 ``Int8`` 的别名，所以还牵扯到 Genrics 类型转换，不够方便。

如果非要作为 CString 处理，可以用 ``reinterpretCast()``，直接转换。但是请一定要知道自己在转换什么，确保类型的 sizeof 相同，确保转换本身有意义。

例如获得环境变量字符串：

```
let key = "PATH"
// 这里相当于把 UnsafePointer<CChar> 转为了 UnsafePointer<UInt8> 然后到 CString
let path_str: CString = reinterpretCast(key.withCString(getenv))
```

### Unmanaged

这个先挖坑，随后填上。

### VaList

这个也是坑，随后填上。

## 第三部分 C 调用 Swift

如果项目里加入了 C 文件，那么它可以调用我们的 Swift 函数么？答案是可以的，而且令人吃惊地透明。这也许是因为 Apple 所宣传的，Small Runtime 概念吧。极小的语言运行时。

和 Objective-C 混编类似，配置好 Bridging Header 的项目，在 .c .h .m 文件中都可以使用一个叫做 ``ProjectName-Swift.h`` 的头文件，其中包含 Swift 代码导出的函数等。

参考之前的 Objective-C 和 C 交互我们可以知道，说到底交互就是链接过程，只要链接的时候能找到符号就可以。

不过不能高兴太早，Swift 是带类、枚举、协议、多态、泛型等的高级语言，符号处理明显要比 C 中的复杂的多，现代语言一般靠 name mangle 来解决这个问题。也就是说一个 Swift 函数，在编译到 .o 的时候，名字就不是原来那么简单了。比如 ``__TFV5hello4Rectg9subscriptFOS_9DirectionSi`` 这样的名字。

Xcode 自带了个工具， 可以查看这些 mangled name 到底是什么东西：

```
xcrun swift-demangle __TFV5hello4Rectg9subscriptFOS_9DirectionSi
_TFV5hello4Rectg9subscriptFOS_9DirectionSi ---> hello.Rect.subscript.getter (hello.Direction) -> Swift.Int
```

当我们从 C 调用的时候，应该规避这样的名字。还记得前面的 ``@asmname`` 么？没错，它可以用于指定 Swift 函数的符号名，我猜测应该有指定 mangled name 的作用，但是不是特别确定。

这里随便指定个例子先。

```scala
@asmname("say_hello") func say_hello() -> Double {
    println("This is say_hello() in swift")
    return 3.14
}
```

然后在 .c 文件中：

```scala
#include <ProjectName-Swift.h>

extern double say_hello();

int some_func() {
  say_hello(); // or capture its value and process it
  return 0
}
```

~~当然这里的 extern 一行是可选的，因为实际上声明会存在于 ProjectName-Swift.h 中，只是为了避免首次编译警告而已（第二次以后的编译，其实这个 Header 已经被缓存起来了，这个牵扯到 Xcode 的编译过程）。~~ 错了。

对于函数而言 extern 必须手动加上，对于 class  、 protocol ，会在生成的头文件里。

按照这个思路，其实很容易实现 Swift 调用 C 中调用了 Swift 函数的函数。这意味着，可以通过简单的方法封装支持向 C 传递 Swift block 作为回调函数。难度中上，对于有过类似扩展编写经验的人来说很简单。

## 第四部分 编译过程

其实调用基本就这么多， Objective-C 那篇文章中说的编译过程同样有效。我 C-c C-v 下：

- 编译所有 ``X.swift`` 文件到 ``X.o`` (with ``-emit-objc-header``, ``-import-objc-header``) (其中包含 ``.swiftmodule`` 子过程)
  - 由于选项里有 ``-emit-objc-header``，所以之后的 C 文件可以直接 import 对应的 ``ProjectName-Swift.h``
- 编译 ``X.c`` 到 ``X.o``
- 链接所有 ``.o`` 生成可执行文件

仔细理解上面的简简单单四行编译过程说明，你就明白为什么 .swfit 和 .c 可以互相调用了。其中两个 Header 文件起到了媒介的作用，一个将 .c/.m 文件中的定义暴露给 Swift，另一个将 .swift 中的定义暴露给 .c/.m 。

