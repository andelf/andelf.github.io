---
layout: post
title: "为第三方扩展创建 Swift 模块"
date: 2015-01-23 23:21 +0800
comments: true
categories: swift swift-module
---

本文提出了一种将第三方扩展引入到 Swift 标准库的方法。

以 Alamofire 为例，

```
cd Path-To-Alamofire-Src-Dir
mkdir -p 32 64

# 创建动态链接库，及对应 Swift 模块，32/64版本
xcrun swiftc -sdk $(xcrun --show-sdk-path --sdk iphoneos) Alamofire.swift -target arm64-apple-ios7.1 -target-cpu cyclone -emit-library -emit-module -module-name Alamofire -v -o libswiftAlamofire.dylib -module-link-name swiftAlamofire -Xlinker -install_name -Xlinker @rpath/libswiftAlamofire.dylib

mv Alamofire.swiftdoc Alamofire.swiftmodule libswiftAlamofire.dylib ./64

xcrun swiftc -sdk $(xcrun --show-sdk-path --sdk iphoneos) Alamofire.swift -target armv7-apple-ios7.1 -target-cpu cyclone -emit-library -emit-module -module-name Alamofire -v -o libswiftAlamofire.dylib -module-link-name swiftAlamofire -Xlinker -install_name -Xlinker @rpath/libswiftAlamofire.dylib

mv Alamofire.swiftdoc Alamofire.swiftmodule libswiftAlamofire.dylib ./64

# 创建 universal lib
lipo -create ./{32,64}/libswiftAlamofire.dylib  -output ./libswiftAlamofire.dylib

# 创建模拟器用 lib
xcrun swiftc -sdk $(xcrun --show-sdk-path --sdk iphonesimulator) Alamofire.swift -target i386-apple-ios7.1 -target-cpu yonah -emit-library -emit-module -module-name Alamofire -v -o libswiftAlamofire.dylib -module-link-name swiftAlamofire -Xlinker -install_name -Xlinker @rpath/libswiftAlamofire.dylib
```

其他相关 target
```
-target armv7-apple-ios7.1 -target-cpu cortex-a8
-target arm64-apple-ios7.1 -target-cpu cyclone
-target i386-apple-ios7.1 -target-cpu yonah
-target x86_64-apple-ios7.1 -target-cpu core2
```

其实你了解 Swift 模块结构的化，应该回想到，将第三方模块创建为 swiftmodule 应该是最靠谱的选择。不过实际操作发现，
编译命令无法很方便地调整，主要是因为 xcodebuild 系统，和编译命令不知道怎么导出。也是略纠结。

实际上，如果使用 Carthage 的话，即把第三方扩展作为 Framework 引入，会导致无法支持 iOS 7，但是 Swift 本身是支持 iOS 7 的，
在编译命令和生成的文件中检查发现，对于 iOS 7，Swift 使用了纯静态模块编译的方法。所以其实我们引入第三方扩展的时候也可以这样做。

以下是静态编译所需命令：

```
xcrun swift -sdk $(xcrun --show-sdk-path --sdk macosx) SwiftyJSON.swift -c -parse-as-library -module-name SwiftyJSON -v -o SwiftyJSON.o

ar rvs libswiftSwiftyJSON.a SwiftyJSON.o
```

如何使用？

将编译结果扔到：

```
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/swift
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/swift_static
```

下对应目录。

然后在 Xcode 里，直接 import。
