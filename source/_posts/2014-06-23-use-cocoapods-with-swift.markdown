---
layout: post
title: "Use CocoaPods with Swift (在 Swift 中使用 CocoaPods） "
date: 2014-06-23 20:15:39 +0800
comments: true
categories: swift swift-module
---
声明： 转载注明我或者 SwiftChina 。请在方便的情况下情尽量告知. [weibo](http://weibo.com/234632333)

本文的发现基于个人研究。请尊重原创。已授权 CocoaChina 转载个人文章。

本文介绍如何在 Swift 项目中使用 CocoaPods 。如果你已经精通 Bridging Header 的方法，请直接跳到 "扩展 CocoaPods" 一节。

## 什么是 CocoaPods

> CocoaPods is the dependency manager for Objective-C projects.
> It has thousands of libraries and can help you scale your projects elegantly. [^1]

从介绍看，它是主要给 Objective-C 项目用的，但是我们可以很容易地混合 Objective-C 和 Swift 到同个项目，从而利用大量的 CocoaPods 库和 Swift 漂亮舒服的语法。

作为 iOS 开发新手，一定是要紧跟前人脚步，学习使用 CocoaPods 。

## 基础用法

这里简单略过，请参考其他无数的文章。

### 安装

系统默认安装，可以参考其他教程[^2] 。在命令行下执行。

    sudo gem install cocoapods

我的环境是 HomeBrew 

```
# 添加 taobao Mirror 不然被墙掉没办法下载
gem sources -a http://ruby.taobao.org/ 
# 安装
gem install cocoapods
# 更新命令
rbenv rehash
# 执行
pod
# 此时一般会下载官方的所有 PodSpec 库，也可以用 pod setup 初始化环境
```

本文不打算在安装部分耗费太多时间。希望看到这里保证你的命令行下有可用的 ``pod`` 命令。

### 使用

假设我们已经有个项目，叫 ProjName ，需要使用一些注明的 CocoaPods 库，比如 AFNetworking[^3]. 

首先，命令行 cd 到我们的项目目录，一般 ls 命令会看到如下几个文件夹：

```
ProjName
ProjName.xcodeproj
ProjNameTests
``` 

赞，就是这里，创建一个 ``Podfile`` 文本文件，写入如下内容

```
platform :ios, "8.0"
pod "AFNetworking", "~> 2.0"
```

一般这么简单的文件都是直接 nano 写。 :)

直接创建 ``Podfile`` ， CocoaPods 会创建一个项目同名的 WorkSpace ，然后添加一个叫 Pods 的项目，这个项目编译结果是一个叫 ``libPods.a``的链接库，
它会添加到我们之前的 ProjName 项目中作为编译依赖。

当然，通过命令行执行 ``pod init`` 也可以自动创建 ``Podfile``，而且可以自动分析当前项目的 target ，相对来说更好，也更优雅。具体请参考官方手册。这样的好处是更细致，还可以区分多个子项目子 target 。原理大同小异。

然后接下来，命令行执行 ``open ProjName.xcworkspace``，注意这个可不是 ``.xcodeproj``，这个是 CocoaPods 为我们创建的一个 WorkSpace ，包含我们之前的项目，和 Pods 依赖。

开始编码过程。直接在代码里调用，比如写在某个按钮的 ``@IBAction`` 里：

```scala
        let manager = AFHTTPRequestOperationManager()
        let url = "http://api.openweathermap.org/data/2.5/weather"
        println(url)

        let params = ["lat": 39.26, "lon": 41.03, "cnt":0]
        println(params)

        manager.GET(url,
            parameters: params,
            success: { (operation: AFHTTPRequestOperation!,
                        responseObject: AnyObject!) in
                println("JSON: " + responseObject.description!)
            },
            failure: { (operation: AFHTTPRequestOperation!,
                        error: NSError!) in
                println("Error: " + error.localizedDescription)
            })
```

这里直接抄了 JakeLin 的 SwiftWeather 代码[^5]，就一小段，希望他不会打我。

#### Swift 坑爹了

看起来貌似我们已经可以在 Swift 中使用 ``AFNetworking`` 了。结果刚写几句代码一堆类和变量找不到定义，而且坑爹的是很多时候我们只能靠猜测，判断这些 Objective-C 的定义转换成 Swift 定义是什么样子，用起来就是完全靠蒙！

这不科学！

这都三礼拜了，所以大家都摸索出了调用的方法，那就是按照和 Objective-C 代码混编的例子，添加 Bridging Header ！

#### 继续

之前简单介绍过和  Objective-C 交互的内容[^4]，大家可以去围观。

一般说来，你在 Swift 项目新建 Objective-C 类的时候，直接弹出是否创建 Bridge Header 的窗口，点 YES 就是了，这时候一般多出来个 ``ProjectName-Bridging-Header.h`` 。然后删掉这个类， Bridging Header 头文件还在。

在这个 Bridging Header 文件里写入要导入的 CocoaPods 库，就可以在 Swift 中使用了。

    #import <AFNetworking/AFNetworking.h>

如果没有自动创建头文件的话，这个配置在项目的 Build Settings 中的 Swift Compiler - Code Generation 子项里。

创建一个头文件，指定为 Bridging Header 也可以。

然后编译，成功执行！

#### 这就完事了？    

实际上，前两天刚写一篇 [Swift 的模块系统](http://andelf.github.io/blog/2014/06/19/modules-for-swift/) ， 把任意 Objective-C 库当做 Swift Module 是可行的。当时就觉得这个东西应该是可能完全进入 CocoaPods 的，但是在官方 repo 找了下发现，以前有人提过增加 ``module.map`` 支持，结果 CocoaPods 的人认为这个是 llvm 内部特性， issue 被关闭了。[#2216](https://github.com/CocoaPods/CocoaPods/issues/2216) 最近又被提起，我在后面提了下 Swift 支持，希望官方靠谱。

所以下面的内容，就是，我们是否可以在 CocoaPods 上加入 ``module.map`` 支持，然后直接在 Swift 中 ``import ModuleName`` ？

## 扩展 CocoaPods

考虑了多种方式，最后选择了 Hook 的方式。如果 Ruby 技术足够好，或许可以直接写个插件。或者直接改官方代码给官方提交。但是实在能力有限。相关的 ``module.map`` 语法参考 llvm 官方手册 [Modules – Clang 3.5 documentation](http://clang.llvm.org/docs/Modules.html)。用了最简单的功能。也许遇到复杂的 PodSpec 就不起作用了，但是原理如此，相信小伙伴们已经知道怎么做了。

目前我的 ``Podfile`` 大概是这个样子：

```ruby
platform :ios, "8.0"
pod "AFNetworking", "~> 2.0"
pod "Baidu-Maps-iOS-SDK", "~> 2.0"

post_install do |installer|
  File.open("#{installer.sandbox_root}/Headers/module.map", 'w') do |fp|
    installer.pods.each do |pod|
      normalized_pod_name = pod.name.gsub('-', '')
      fp.write <<EOF
module #{normalized_pod_name} [system] {
  umbrella "#{pod.name}"
  export *
}
EOF
      puts "Generating Swift Module #{normalized_pod_name.green} for #{pod} OK!"
    end
  end
end
```

``post_install`` 是 ``Podfile`` 的一种 hook 机制，可以用来加入自定义操作。我在这里的写的逻辑就是，针对所有的 Pod 生成一个 ``module.map`` 文件。
位于 ``Pods/Headers/``，这个目录被 CocoaPods 自动设置为项目的 Header Search Path 所以不需要额外处理。默认我们的 Swift 文件就找得到。

其中 ``normalized_pod_name`` 用于处理百度地图 API SDK 这一类名字带减号的库，因为他们不能作为 Module Name ，实际上或许有更好的方法来处理。

### 实际效果

实测发现完全没有问题，直接 ``import AFNetworking`` 或者 ``import BaiduMapsiOSSDK`` 都可以。

而且很不错的一点是，按住 Command 键，然后鼠标点击模块名、类名等，会跳转到 Swift 定义。

### 坑

遇到提示 ``.pcm`` 文件 outdate 的情况下需要你删除 ``$HOME/Library/Developer/Xcode/DerivedData/ModuleCache`` 目录，这个目录保存的是预编译模块，类似于预编译头文件。

目前 Swift 还是有很多 BUG 的，调用 ``NSObject`` 也许会让编译器直接 segment fault ，不带任何出错信息。很伤情。此时请第一时间检查语法是否有诡异，其次将所有用到字符串或者 ``Optional`` 的地方都额外用变量处理，避免用字面常量。（个人经验）

如果多次调用 ``pod install`` 并在其中修改过 ``Podfile``，那么有可能你的项目依赖会乱掉，多了不存在的 ``.a`` 文件到依赖或者多次包含。手工在项目树和项目选项里删除就可以了。此类编译错误都是链接错误。

## 总结

本文提出了一种 Bridging Header 之外的使用 CocoaPods 库的方法。利用有限的 Ruby 知识写了个 Hook 。目前测试 OK 。

## 参考

[^1]: [CocoaPods Offical Site](http://cocoapods.org/) CocoaPods 官网
[^2]: [CocoaPods - CocoaChina](http://www.cocoachina.com/applenews/devnews/2014/0623/8917.html) CocoaChina 对 CocoaPods 的介绍
[^3]: [AFNetworking - Github](https://github.com/AFNetworking/AFNetworking)
[^4]: [Swift and ObjectiveC Interop (Swift 与 Objective-C 之间的交互)](http://andelf.github.io/blog/2014/06/11/swift-and-objectivec-interop/)
[^5]: [SwiftWeather](https://github.com/JakeLin/SwiftWeather/blob/master/Swift%20Weather/ViewController.swift)
