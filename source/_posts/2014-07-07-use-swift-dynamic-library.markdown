---
layout: post
title: "Use Swift Dynamic Framework （如何科学地引用第三方 Swift 库)"
date: 2014-07-07 12:09:54 +0800
comments: true
categories: swift swift-module
---

排名 16 了。啧啧。你看才刚出一个月。

目前已经有了很多非常棒的 Swift 第三方库， JSON 处理啊、 HTTP 访问啊、 UIView 插件啊等等。

如何科学地引用这些第三方库呢？

## 现状

CocoaPods 由于完全使用静态链接解决方法，过度依赖 Objective-C ，目前应该是官方 repo 有提到是 ``-Xlinker`` error , 这个问题之前我也遇到过，无解。除非手工执行 ``ar`` 不用 ``ld`` 和 ``libtool``。

小伙伴有用子目录的方法引用代码，貌似不错，还有就是直接用 ``git submodule``，看起来维护性也可以。

## 简单解决方案 

一个良好的第三方库应该实现为 Cocoa Touch Framework (实际内容为 Header + 动态链接库)。而不是直接把 Swift 代码 Copy 过来放入自己的项目。这里以一个简单项目为例，介绍如何科学使用。

### 目标描述

用 Swift 创建一个 Demo ，使用 SwiftyJSON 和 LTMorphingLabel 库。

项目的名字叫 DemoApp 。

#### 创建 Workspace

创建一个 Workspace ，名字随意，位置能找到就好。这个 Workspace 主要用来管理我们的项目及其依赖的第三方库。

#### 创建 DemoApp

在 Workspace 创建一个 App ，因为是测试所以我选了 Single View Application 。

#### 引入 SwiftyJSON

SwiftyJSON 是一个 Cocoa Touch Framework ，可以直接使用， ``git clone`` 后，添加项目到 Workspace 即可。

尝试操作发现。。最容易最不会出错的方法就是直接从 Finder 里把 ``.xcodeproj`` 文件拖动到 Workspace 。

#### 引入 LTMorphingLabel

LTMorphingLabel 是一个 App Deme 式项目。其中 Label View 的实现在一个子目录中。可以采用创建 Cocoa
Touch Framework 的方法来引入这几个文件。

当然也可以直接把目录拖到我们的 DemoApp 里，不过太原始粗暴了。

#### 为 App 添加依赖 

在 DemoApp 的 Genral 选项卡中，添加 Linked Frameworks and Libraries 。选择  Workspace 中 SwiftyJSON 和
LTMorphingLabel 两个 ``.framework`` 。

如果是直接选择来自其他项目的 ``.framework`` 而不是同一 Workspace ，那么这里也许还要同时加入 ``Embedded Binaries``。

#### 使用

添加好依赖后，就可以在 DemoApp 项目代码中 ``import SwiftyJSON`` 或者 ``import LTMorphingLabel`` 来使用对应的库。同时还可以用 Command + 鼠标点击的方法查看声明代码。

#### 除错

比较坑爹的是，实际上按照以上方法， ``LTMorphingLabel`` 并不能正常使用，查看报错信息发现是自动生成的 ``LTMorphingLabel-Swift.h`` 有处语法无法被识别，编辑器找到 ``.h`` 文件，注释掉这行诡异代码即可。

看起来目前的 Bridge Header 和 -emit-objc-header 实现还是有问题的。小伙伴一定要淡定。

## 对于非 Workspace

如果不喜欢使用 Workspace ，也可以将第三方库的编译结果，一个 ``.framework`` 目录拖到项目文件里，然后添加 ``Embedded Binaries``。

## 评论

创建 Cocoa Touch Framework 选项中，可以使用 Swift 代码，此时编译结果（默认）会包含 ``module.modulemap`` 文件，
之前有介绍过它的作用，通过它， Swift 可以使用第三方模块。参考 [Module System of Swift (简析 Swift 的模块系统)](http://andelf.github.io/blog/2014/06/19/modules-for-swift/) 。

实际上这个解决方案绕了一大圈，通过 Swift 文件导出 ``ProjName-Swift.h``、然后 ``module.modulemap`` 模块描述文件引入、然后再由 Swift 导入。

其实 ``.framework`` 同时也包含了 ``ProjName.swiftmodule/[ARCH].swiftmodule`` 不过看起来没有使用到，而且默认在 IDE 下也不支持 Swift 从 ``.swiftmodule`` 文件导入，比较坑。希望以后版本能加入支持。

``.framework`` 包含了所有 Swift 标准库的动态链接库，小伙伴可能会以为这会导致编译后的 App 变大。其实大可放心，任何 Swift 语言的 App 都会包含这些动态链接库，而且只会包含一个副本。此方法对 App 最终的大小几乎无影响。

注： 个人测试了下，发现这个 ``.swiftmodule`` 是可以通过其他方法使用的，绕过 ``module.modulemap``，应该是更佳的解决方案，但是需要控制命令行参数。

至于静态链接库，过时了。抛弃吧。

## 参考

- [Module System of Swift (简析 Swift 的模块系统)](http://andelf.github.io/blog/2014/06/19/modules-for-swift/)
- [Write Swift Module Cont. Static Library （使用 Swift 创建 Swift 模块 - 静态链接库）](http://andelf.github.io/blog/2014/06/25/write-swift-module-with-swift-cont/)
