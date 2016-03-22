---
layout: post
title: "Swift Attributes"
date: 2014-06-06 17:12:53 +0800
comments: true
categories: swift
---

## @availability(*, unavailable, message="a Message")
to disable a func?

## @asmname("a_name")

Undocumented Attributes By me. :)

@asmname("swift_xxxx") func funname(...)

Builtin asm symbol. maybe internal functions.

这里用于调用 C 函数。相当于与 Objective-C 交互时候用的 @objc(name)。避免名字冲突，也可以相当于别名。

使用方法：

```c
int println() { .... }
```

```
@asmname("println") func c_println() -> CInt; // 声明
c_println() // 调用
```

也就是 C 中的同名函数，我们可以给赋予一个别名，然后正常调用。

同时也用于指定 Swift 导出符号的名字，用于在 C 语言调用。

详细参考 [简析Swift和C的交互](http://andelf.github.io/blog/2014/06/15/swift-and-c-interop/)

## @transparent
用在函数、方法定义前，根据 lldb 和 llvm ir 猜测，作用相当于 inline。内联函数。

需要扯一点 Swift 的编译，会生成 ``.swiftmodule``，这个文件里包含信息更多，引用外部 ``.swiftmodule`` 时候，inline 仍然有效。

## @assignment
用于重载操作符时表示赋值副作用。参数类型需要标记为 ``inout``。参考运算符重载的那篇文章。

[Swift 运算符重载、自定义运算符](http://andelf.github.io/blog/2014/06/06/swift-operator-overload/)

## @final 
这个好像没什么多说的。文档里有。类似 java 的 ``final``，C井中的 ``sealed``。

## @conversion
参考上面的一篇。隐式类型转换好了。

[Implicit Type Convertion in Swift //Swift 的隐式类型转换](http://andelf.github.io/blog/2014/06/08/swift-implicit-type-cast/)

## @noreturn 
表示函数不返回它的调用者。类似于 Rust 的 Buttom Type。

实例：

```scala
@noreturn @transparent func fatalError(message: StaticString, file: StaticString = default, line: UWord = default)
```

执行后打印出错消息，退出。

## @auto_closure
用于函数参数类型前，表示一个空参数的 closure，调用时直接以语句的方式创建 closure，很自然。

```scala
func simpleAssert(condition: @auto_closure () -> Bool, message: String) {
    if !condition() {
        println(message)
    }
}
let testNumber = 5
simpleAssert(testNumber % 2 == 0, "testNumber isn't an even number.")
// prints "testNumber isn't an even number.
```

``simpleAssert`` 中第一个参数，实际上是自动创建了一个 closure。文档中有详细描述。

## @IBAction, @IBDesignable, @IBInspectable, and @IBOutlet
同 Objective-C

## @objc @objc(name:name:)
参考文档。

## @objc_block

## @class_protocol

## @required

## @optional

## @UIApplicationMain

## @NSManaged

## @NSCopying

## @lazy

In class property:

    @lazy let prop_name: PropType = PropType()

## @exported

e.g.

@exported import ModName

## @sil_weak
## @sil_unowned
## @sil_unmanaged

猜测是语言内部表示一个属性禁用 ARC？

Safety Integrity Level

## @sil_self

## @opened(...)

## @cc(...)

## @autoreleased


## @objc_metatype

## @LLDBDebuggerFunction
## @requires_stored_property_inits

```
@requires_stored_property_inits class Point {
  ...
```
## only in sil modules

- @callee_unowned
- @callee_owned
- @callee_guaranteed
- @guaranteed
- @out
- @thin
- @thick
- @owned

## unused 

这里可以看到一些 Swift 的历史变迁。4 年也不算很短。

- @lvalue
- @unchecked
  - '@unchecked T?' syntax is going away, use 'T!'
- @inout
  - @inout is no longer an attribute 应该是现在的 inout 关键字
- @unowned
  - '@unowned' is not an attribute, use the 'unowned' keyword instead
