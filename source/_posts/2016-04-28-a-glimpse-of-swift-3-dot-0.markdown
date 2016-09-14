---
layout: post
title: "Swift 3.0 尝试——从入门到再学一门(A Glimpse of Swift 3.0)"
date: 2016-04-28 10:06:53 +0800
comments: true
categories: swift
---

安装工具

https://github.com/kylef/swiftenv

## Swift 3.0 新变化

以下内容来自 Swift 语言提案[^1]。

[^1]: (apple/swift-evolution)[https://github.com/apple/swift-evolution]

Swift 3.0 发布计划

### Swift Package System

https://github.com/donald-pinckney/swift-packages


## AST 结构

代码位于 ``include/swift/AST`` 和 ``lib/AST``。

ModuleDecl 
模块（单个库或是可执行文件）。编译的最小单元，由多个文件组成。

FileUnit（抽象类）
文件作用域，是代码组织的最小单元。

- DerivedFileUnit: A container for a module-level definition derived as part of an implicit protocol conformance.
- SourceFile: A file containing Swift source code. .swift 或 .sil 也可以是虚拟 REPL
  - Imports: Vec<(ImportedModule, ImportOptions)>
  - Identifier
  - Decls: Vec<Decl>
  - LocalTypeDecl: Vec<TypeDecl>
  - ObjCMethods: Map<ObjCSelector, AbstractFunctionDecl>
  - infix, postfix, prefix operators: OperatorMap
- BuiltinUnit


## swift 命令入口

入口函数 ``tools/driver/driver.cpp``，``main`` 函数。

集成多个子工具。同时若 ``PATH`` 下有名为 ``swift-foobar`` 的可执行文件，则可通过 ``swift foobar`` 调用。

### 编译器前端 ``swift -frontend``

编译。同时支持打印出各种编译时中间结果。

### API Notes 功能 ``swift -apinotes``

参考信息位于 [https://github.com/apple/swift/tree/master/apinotes](https://github.com/apple/swift/tree/master/apinotes) .

简单说，API Notes 机制就是通过 ``.apinotes`` 文件（YAML格式）描述 Objective-C Framework 和对应 Swift API 的关系。最终生成 ``.apinotesc`` 文件，与 ``.swiftmodule`` 文件一起作为 Swift 的模块。

主要功能包括且不限于：

- ``SwiftBridge``：设置对应的 Bridge 类型，例如 NSArray 对应与 ``Swift.Array``
- ``Nullability``/``NullabilityOfRet``： 类的属性、方法的参数、返回值对应类型是否可以为 null，即对应与 Swift 的 T 还是 T? 
- ``Availability``：方法是否在 Swift 中暴露，并给出 availability message
- ``SwiftName``：方法 Selector 在 Swift 中的重命名，例如 ``filteredArrayUsingPredicate:`` 替换为 ``filtered(using:)``

Dump 为YAML文件：

    $> swift -apinotes -binary-to-yaml /path/to/lib/swift/macosx/x86_64/Dispatch.apinotesc -o=-


### Module Wrap 工具 ``swift -modulewrap``

```
// Wraps .swiftmodule files inside an object file container so they
// can be passed to the linker directly. Mostly useful for platforms
// where the debug info typically stays in the executable.
// (ie. ELF-based platforms).
```

用法：

    swift -modulewrap ObjectiveC.swiftmodule -o objc.o

实际发现是在 ``.o`` 里定义了 ``___Swift_AST`` 符号。

### REPL

Swift 提供了两个 REPL(Read-Evaluate-Print Loop)，一个是 Swift 本身内置，另一个集成到了 lldb 命令行下。前者只有基本功能，即将废弃，后者功能更强大。

子命令分别是：

- ``swift -deprecated-integrated-repl``
- ``swift -lldb-repl``。

``swift -repl`` 子命令选择可用的 REPL 进入，一般是 ``lldb-repl``，除非找不到 lldb 时。这也是 Swift 命令不带任何参数的默认行为。


