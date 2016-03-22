---
layout: post
title: "NSObject Pattern Match in Swift (简析 Swift 中的 Pattern Match)"
date: 2014-06-17 09:29:48 +0800
comments: true
categories: swift
---
本文正式标题是：简析 Swift 中的 Pattern Match

副标题是：妈蛋 Swift 你又用黑属性坑大家了: 为什么 switch case 可以用数字匹配字符

声明： 原创发现，原创内容。转载注明我或者 SwiftChina . [weibo](http://weibo.com/234632333)

## 问题的提出

故事是这样的，昨天有人在论坛上发了个帖子，虽然不太规范而且含糊，不过还是能看出来他在问什么。[传送门](http://swift.sh/topic/151/switch-case10/)

重新把问题简化搬运过来。

```scala
let count = "0"
switch count {
    case 1:
    println("location 1")
    case 0:
    println("location 2")
    case "1":
    println("location 3")
    default:
    println("location 4")
}
```

为什么上面的代码可以通过编译？

## 问题验证

这几天说了很多了， Swift 是强类型语言，所以类型不同肯定不能在一起。直觉上，这段代码肯定不可能编译通过，错误一定是类型错误。

果断扔代码到文件，然后编译（我个人习惯用命令行直接编译）:

```
// 填入 test.swift
xcrun swift -g test.swift
```

果然出错，不过出错信息略诡异：

```
test.swift:4:6: error: could not find an overload for '~=' that accepts the supplied arguments
case 1:
     ^
```

故事到这里也许完事了。嗯，不可以编译，问题也许是 Xcode 6 beta 版的 BUG 什么的。微博上的 [@肇鑫](http://weibo.com/owenzx) 也提到， 用 Playground 编译也是出错，出错信息同上。

看起来暂时是无法重现问题。

### 出错信息

先解释下这个出错信息，熟读那本 ibook 的同学，可能会记得，在 swift case 那节是有讲到 ``~=`` 这个运算符的，也讲到了如何自定义它。

> Expression Pattern

> An expression pattern represents the value of an expression. Expression patterns appear only in switch statement case labels.

> The expression represented by the expression pattern is compared with the value of an input expression using the Swift standard library ~= operator. The matches succeeds if the ~= operator returns true. By default, the ~= operator compares two values of the same type using the == operator. It can also match an integer value with a range of integers in an Range object.

所以其实 ``~=`` 是一个匹配运算符(Pattern Match Operator)，专门用在 Pattern Match 的地方，比如 switch 的 case 。一般情况下它直接调用 ``==`` 运算符。

原书还写道：

> You can overload the ~= operator to provide custom expression matching behavior. For example, you can rewrite the above example to compare the point expression with a string representations of points.

说明 Pattern Match 的行为是可以通过重载 ``~=`` 控制的。书中有重载的例子，小伙伴们可以看看。

深入下，来看看 ``~=`` 到底是什么东西：

```
@transparent func ~=<T : Equatable>(a: T, b: T) -> Bool
func ~=<T : RandomAccessIndex where T.DistanceType : SignedInteger>(x: Range<T>, y: T) -> Bool
```

可以看到，首先它可以匹配两个相同类型的 Equatable，而且是 ``@transparent``，说明没有额外开销， inline 函数，从书中看，完全等价于调用 ``==``，不深究了。

然后，它可以匹配一个 ``Rnage<T>`` 和 ``T``，这也就是我们在 switch case 里写的 ``case 1..18: println("未成年")`` 背后的奥秘，略有开销。

以上说明了为什么出错信息提示为什么是 ``~=`` 运算符重载失败，而不是类型错误（其实也算是一种类型错误，没有找到适合运算符的类型）。

### 再次尝试重现

有了上面的线索，是不是怀疑我们遗漏了什么？回到原问题提出的地方：

```scala
override func viewDidLoad() {
super.viewDidLoad()
let count = "0"
....
```

看起来，他是在 App 环境下调用的，这也许就是问题的根源。之前有介绍过， Cocoa 框架给 Swift 语言添加了很多东西，包括一大批的 ``extension``，还有一堆隐式类型转换。

我们在 ``test.swift`` 第一行加入 ``import Cocoa``，重新编译。结果正常通过编译。代码执行结果符合预期，走 default 分支。

### 这也不科学!

小伙伴们惊呆了，看起来 ``count`` 变量明明是一个 ``Character``，结果竟然能匹配 ``Character`` 和 ``Int``。

## 祭出 lldb

lldb 挂上 exe ，看看汇编代码是怎么样的。带上注释。我摘录最能说明问题的一段。

```gas
   ; 这里是 case 1
   0x100001d1a:  movabsq $0x01, %rdi
   0x100001d24:  movq   %rax, -0x148(%rbp)

   ; 将 1 转换为 NSNumber
   0x100001d2b:  callq  0x100003f32               ; symbol stub for: Swift.Int.__conversion (Swift.Int)() -> ObjectiveC.NSNumber

   ; 此处省略调用 ARC 代码若干
   ....

   ; 这里其实是 count: String , 三个寄存器表示
   ; 检查 String 大小你会发现 sizeof(String) 是 24，三个 64bit
   ; 熟悉 Rust 的同学知道，字符串实现，无非三个 field ： size 、 capacity 、pointer
   ; 猜测 Swift 完全相同因为它大量借鉴了 Rust （不服来喷）
   0x100001d43:  movq   -0x128(%rbp), %rdi
   0x100001d4a:  movq   -0x130(%rbp), %rsi
   0x100001d51:  movq   -0x138(%rbp), %rdx

   0x100001d58:  movq   %rax, -0x158(%rbp)
   ; 这里将 count: String 转换为 NSString
   0x100001d5f:  callq  0x100003f26               ; symbol stub for: Swift.String.__conversion (Swift.String)() -> ObjectiveC.NSString
   0x100001d64:  movq   -0x150(%rbp), %rdi
   0x100001d6b:  movq   %rax, %rsi
   ; **最最神奇的地方来了，这里调用了 ~= ObjectiveC 版，比较两个 NSObject 的版本。
   0x100001d6e:  callq  0x100003f20               ; symbol stub for: ObjectiveC.~= @infix (ObjectiveC.NSObject, ObjectiveC.NSObject) -> Swift.Bool
```

## 分析

上面一堆看不懂没关系，这里再复述下在 Cocoa 环境（或者说 Foundation 环境下）第一条 ``case 1:`` 的流程:

- 将 1 转换为 ``NSNumber``，通过隐式类型转换实现
- 将 ``count`` 转换为 ``NSString``，通过隐式类型转换实现
- 调用 ``NSObject`` 重载版的 ``~=`` 运算符进行比较

这么看起来明了多了。但是，我们的 ``Character`` 哪里去了？为什么 count 的类型变了？

直接下断点然后 repl 命令检查，果然，我们没有指定类型的 ``count`` 在这个版本中变成了一个 ``String``。

从标准库定义看， ``Character`` 和 ``String`` 都实现了 ``ExtendedGraphemeClusterLiteralConvertible`` 协议，保证他们能从字符字面常量获得。所以实际上每当我们写 ``let foo = "a"`` 其实是完全依赖编译器的类型推导，选择到底是 ``Character`` 还是 ``String``。

说明 Swift 编译器高大上，可以从当前加载的所有类型中推导出合法合理能编译的类型组合。来自 Haskell 背景的同学一定笑而不语，当然那种怎么改都凑不对类型的痛苦也是其他同学无法理解和感同身受的。

回头再看看 ``ObjectiveC.~=`` 这个运算符重载：

```scala
func ~=(x: NSObject, y: NSObject) -> Bool
```

定义在 ObjectiveC 模块里，使用 ``import ObjectiveC`` 可以加载。这个模块是 Swift 到 Objective-C 的一些映射模块，基础的 ``NSObject`` 方法还有一些运行时控制函数。

当然 ``Foundation``, ``Cocoa`` 这些都会加载它，所以理论上写应用的时候不需要指定，只要你用到了 ``NSObject`` 就一定有它。

实际上 WWDC 某一集里讲到（忘记了，等我再找到会补上），对比 ``NSObject`` 的时候，Swift 会选择调用 ``isEqual`` 方法。我们反汇编验证下。简单加点注释。

```gas
(lldb) dis -a 0x100003f20
asmtest`symbol stub for: ObjectiveC.~= @infix (ObjectiveC.NSObject, ObjectiveC.NSObject) -> Swift.Bool:
   0x100003f20:  jmpq   *0x1252(%rip)             ; (void *)0x00000001000e88d0: ObjectiveC.~= @infix (ObjectiveC.NSObject, ObjectiveC.NSObject) -> Swift.Bool
(lldb) dis -a 0x00000001000e88d0
libswiftObjectiveC.dylib`ObjectiveC.~= @infix (ObjectiveC.NSObject, ObjectiveC.NSObject) -> Swift.Bool:
   0x1000e88d0:  pushq  %rbp
   0x1000e88d1:  movq   %rsp, %rbp
   0x1000e88d4:  pushq  %r15
   0x1000e88d6:  pushq  %r14
   0x1000e88d8:  pushq  %rbx
   0x1000e88d9:  pushq  %rax
   ; 64位 寄存器传参
   0x1000e88da:  movq   %rsi, %rbx
   0x1000e88dd:  movq   %rdi, %r14
   0x1000e88e0:  movq   0x3da1(%rip), %rsi        ; "isEqual:"
   0x1000e88e7:  movq   %rbx, %rdx
   0x1000e88ea:  callq  0x1000eb670               ; symbol stub for: objc_msgSend
   ; 测试 isEqual: 调用的返回值，存到 %r16b
   0x1000e88ef:  testb  %al, %al
   0x1000e88f1:  setne  %r15b
   0x1000e88f5:  movq   %rbx, %rdi
   0x1000e88f8:  callq  0x1000eb676               ; symbol stub for: objc_release
   0x1000e88fd:  movq   %r14, %rdi
   0x1000e8900:  callq  0x1000eb676               ; symbol stub for: objc_release
   ; 本运算符函数返回值
   0x1000e8905:  movb   %r15b, %al
   0x1000e8908:  addq   $0x8, %rsp
   0x1000e890c:  popq   %rbx
   0x1000e890d:  popq   %r14
   0x1000e890f:  popq   %r15
   0x1000e8911:  popq   %rbp
   0x1000e8912:  retq
```

啧啧， Instruction Pointer Relative Addressing 用的飞起。 64 位名字也特好听。 RIP 。 对了， MacOSX 下都是 PIC 位置无关代码。

从上面看，果然是调用了 Objective-C 的 ``isEqual:``。

## 总结

- Swift 的编译器会根据大范围的代码推导类型，遍历 AST 然后用树或者图算法填充未指定的类型
  - ``"c"`` 可以是任意 ``ExtendedGraphemeClusterLiteralConvertible`` 类型，包括 ``Character``、``String``、``CString``、``UnicodeScalar``、``StaticString`` 等
- swith case 语句使用 ``~=`` 运算符执行匹配操作
  - 非 Foundation 环境下，相当于 ``==`` 运算符
  - 在 Foundation 环境下，如果是 ``NSObject`` 及其子类，相当于 ``isEqual:`` 方法
  - ``a..b`` 这样的匹配也是通过 ``~=`` 运算符实现，也可以重载后自定义
- 在 Foundation 环境下，隐式类型转换遍地都是

## 教训

- 真用到字符类型 ``Character`` 的时候，还是显式指定吧
- 需要额外注意能隐式类型转换到 ``NSObject`` 的几个类型，避免非预期行为

## 参考文献

- Apple Inc. “The Swift Programming Language”。 iBooks. https://itun.es/cn/jEUH0.l
- [我的 Github andelf/Defines-Swift](https://github.com/andelf/Defines-Swift)
- [隐式类型转换](http://andelf.github.io/blog/2014/06/08/swift-implicit-type-cast/)
- WWDC 某集讲到的 ``isEqual:``

