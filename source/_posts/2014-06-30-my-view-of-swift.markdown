---
layout: post
title: "My View of Swift （闲扯对 Swift 语言的看法）"
date: 2014-06-30 11:05:13 +0800
comments: true
categories: swift
---

其实这是很早就像要说的了，大概当时信誓旦旦说要看完那本 epub 写个读后感谈谈对 Swift 看法什么的。后来不了了之。
现在觉得这个时机或许差不多，对 Swift 的了解也算凑合了。

纯个人观点。

### Swift 是系统编程语言

一开始大家还不太了解的时候，可能会有很多误解。现在好歹一个月了。误解终于少了。

是的， Swift 是系统编程语言，原因是因为它 ABI 兼容 C （不包括 name mangling 部分）。基于强大的 llvm 生成具体平台代码。不是翻译为 Objective-C 的。

编译器参数还显示， Swift 文件的中间编译结果（介于 Swift 代码和 llvm ir ）是 SIL ，猜测是 Swift Intermediate Language 。好像和 llvm ir 有所联系。而且至少有两个 stage 。

不是脚本语言，也不是胶水语言。但是它的标准库 (import Swift 库) 几乎不含任何 IO 网络 等内容，随便做个功能强依赖 Cocoa 框架。也可以 ``import Darwin`` 用 C 语言的标准库来写。

猜测写个 Python C 模块这种任务是可以轻易胜任的。

而 Golang 、 Rust 本身 ABI 是和 C 不兼容的。虽然 Rust 通过 ``extern "C"`` 可以修改单个函数为兼容。

### Swift 是现代语言

自动类型推导、泛型、 LLVM 。当然语言研究党都知道这些都是几十年前的“新东西”。

### Swift 是半完成品

这么说主要是指 Swift 对 Cocoa 的库实在是太半吊子了。只是 Foundation 有 Bridge 支持，其他库中，明显的列表都无法支持 subscript 、 for-in 这样简单的操作。原因很简单，这些库都是自动转换 ObjC 头文件而来（参考模块那篇文章）。没有额外的封装代码。

所以其实真要用起来，可能会很纠结。或者可以预计很快就有第三方的 Bridge 库给这些类型加上舒服的 Swift 支持。

另外命令行没有静态链接库支持。只能用其他命令拼装。也侧面说明， Apple 希望开发者更多用动态链接库， Framework 。

另外目前的编译器 coredump 、 stackoverflow 太多太多。错哪都不知道。

### Swift 隐藏细节太多

就对应到 Foundation 类型这个特性太说，太多黑魔法，隐式类型转换、 BridgeToObjectiveC 协议、指针类型转换。

这些隐藏的特性多少都会成为 Swift 的坑。

要知道定义在 ObjC 库的 ``NSString`` 参数某些情况下在 Swift 中被转换为 ``String``。 ``NSArray`` 都被转换为 ``AnyObject[]``。即使有隐式类型转换，某些极端情况下，还是会有编译时错误。

### Swfit 的性能

我没做过测试，但就语言特性来说， Swift 是比 ObjC 快的，因为静态类型使得他在编译时就已经知道调用函数的具体位置。而不是 Objective-C 的消息发送、 Selector 机制。

目前来看， Swift 性能略差原因主要是编译器还没足够优化、还有就是 Cocoa 拖了后腿， Cocoa 本身有大量的 ``AnyObject`` 返回值。所以实际写 Swift 代码时，多用 ``as`` 一定是好习惯。明确类型。

### Swift 的未来

我不知道。至少好像感觉很多培训机构都看到了前途开始疯狂的做视频。

倒是觉得什么时候 Cocoa for Swift 出了才算它完全完成任务。

总觉得 Cocoa 拖后腿，不然放到其他平台也不错。

对了，之前不是在 App 开发领域，这才知道原来这个地盘水很深，太多唯利的培训机构，太多嗷嗷待哺等视频教程的新人。觉得挺有意思。就拿 ? ! 这个 Optional 为例，太多介绍的文章。可惜能说明白的太少太少。糊里糊涂做开发就是当前现状吧。

