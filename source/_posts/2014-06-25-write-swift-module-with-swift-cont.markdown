---
layout: post
title: "Write Swift Module Cont. Static Library （使用 Swift 创建 Swift 模块 - 静态链接库）"
date: 2014-06-25 21:20:55 +0800
comments: true
categories: swift swift-module
---
声明： 转载注明我或者 SwiftChina, 请在方便的情况下情尽量告知. [weibo](http://weibo.com/234632333)

本文的发现基于个人研究。请尊重原创。

## 摘要

本文提出了一种可以编译 Swift 静态链接模块的方法，通过对 swift 编译命令行参数的控制，生成可以自由分发的静态链接库和 swift module 描述文件。同时还提出了导出 objC 头文件供 Objective-C 调用的可能。

关键词： Swift 模块 静态链接库

上次一篇文章 [Module System of Swift (简析 Swift 的模块系统)](http://andelf.github.io/blog/2014/06/19/modules-for-swift/) 中提到：

> 静态链接库 .a 目前还没有找到方法， -Xlinker -static 会报错。

最近摸索了下用 Swift 创建静态链接库的方法。有所收获，这里记录下。

## 废话

我们中的很多人都知道，编译器编译的最后一个步骤一般都是链接，一般都是调用 ``ld``。经过仔细分析，之前为什么不能生成 ``.a`` 静态链接库的原因，发现有如下问题：

- ``-Xlinker -static`` 参数传递的时候， swift 命令本身不能识别，讲 ``-dylib`` 与 ``-static`` 一起传递（这倒不是问题，参数优先级，静态盖掉了动态）
- 链接到 ``-lSystem`` 时候，这个库没有静态链接。

所以总会报错。

### 思考

实际上之前的方法是走了弯路，根本没有必要去调用 ``ld``，作为一个合格的 ``.a`` 静态链接库，只要有对应的 ``.o`` 就可以了，没必要去链接 ``-lSystem``，也许是 swift 本身没有编译为静态链接库的参数支持。

检查 Swift 标准库中的静态链接库，果然只包含对应 ``.swift`` 代码编译后的 ``.o`` 文件。（检查方法是用 ``ar -t libName.a``）

说到底， Swift 静态链接库的目标很简单，就是包含对应 Swift 模块的所有代码，这样就避免了对应动态链接库的引入。和什么 ``-lSystem`` 没啥相干。

## 解决方法 HOWTO

以 lingoer 的 [SwiftyJSON](https://github.com/lingoer/SwiftyJSON) 为例。

我们的目标很简单，就是生成 ``ModName.swiftmodule``、``ModName.swiftdoc``(可选)、``libswiftModName.a`` 三个文件。

### 编译

#### 生成 ``.swiftmodule`` ``.swiftdoc``

```
xcrun swift -sdk $(xcrun --show-sdk-path --sdk macosx) SwiftyJSON.swift -emit-library -emit-module -module-name SwiftyJSON -v -o libswiftSwiftyJSON.dylib -module-link-name swiftSwiftyJSON
```

#### 生成 ``.o``

```
xcrun swift -sdk $(xcrun --show-sdk-path --sdk macosx) -c SwiftyJSON.swift -parse-as-library -module-name SwiftyJSON -v -o SwiftyJSON.o
```

#### 生成 ``.a``

```
ar rvs libswiftSwiftyJSON.a SwiftyJSON.o
```

大功告成。

同时应该也可以用 ``lipo`` 来合成不同平台下的 ``.a`` 链接库。

### 使用

和静态链接库类似，需要 ``-I`` 包含 ``.swiftmodule`` 所在目录， ``-L`` 包含 ``.a`` 所在目录。

如果动态链接库和静态链接库两者同时存在，可以依靠不同目录来区分。

## 你丫闲的！

可能不少人要群嘲，你这意义是啥。你丫闲的。

其实在分发 library 的时候，很多时候我们需要二进制分发，希望别人可以方便地使用。这种情况下，静态链接更佳（虽然新的 iOS 8 支持动态链接，但是看起来是基于 Framework 的，略复杂些。）

甚至我们可以用 ``lipo`` 创建全平台可用的静态链接库。多赞。

## 补充

多个 Swift 文件可以分别编译为 ``.o`` 然后用 ``ar`` 合并。

对于 CocoaPods ，也许可以按照这个逻辑将 Swift 模块暴露出去。需要多加一个参数 ``-emit-objc-header`` （以及 ``-emit-objc-header-path``）即可。

## 参考文献

- 我的另一篇 [Module System of Swift (简析 Swift 的模块系统)](http://andelf.github.io/blog/2014/06/19/modules-for-swift/)
- [Use CocoaPods With Swift (在 Swift 中使用 CocoaPods）](http://andelf.github.io/blog/2014/06/23/use-cocoapods-with-swift/)

