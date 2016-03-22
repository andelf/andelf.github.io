---
layout: post
title: "Swift Misc"
date: 2014-06-13 13:53:13 +0800
comments: true
categories: swift
---

Just Notes. Keep Updating.

## escaped variable name

```
let `let` = 1000
dump(`let`, name: "variable named let")
```

## Optimise Parameter

- ``-Onone`` // optimization off, safety checks on
- ``-O`` // optimization on,  safety checks on
- ``-Ofast`` // optimization on,  safety checks off

## High Level Optimization

- Removing abstraction penalties
- Generic specialization
- Devirtualization (Resolving dynamic method calls at compile-time)
  - If Swift can see where you constructed the object
  - If Swift knows that a class doesn't have any subclasses
  - If you've marked a method with the @final attribute
- ARC optimization
- Enum analysis
- Alias analysis
- Value propagation
- Library optimizations on strings, arrays, etc.


## Closure Memoize

```scala
func memoize<T: Hashable, U>( body: (T)->U ) -> (T)->U {
  var memo = Dictionary<T, U>()
  return { x in
    if let q = memo[x] { return q }
    let r = body(x)
    memo[x] = r
    return r
} }

func memoize<T: Hashable, U>( body: ((T)->U, T)->U ) -> (T)->U {
  var memo = Dictionary<T, U>()
  var result: ((T)->U)!
  result = { x in
    if let q = memo[x] { return q }
    let r = body(x)
    memo[x] = r
    return r
}
  return result
}
```


## lldb

### Raw Display

(lldb) fr v -R age

### Protocol

fr v val

fr v -d r val

### misc

repl // repl loop

// find stop reason
t i // thread info
// how did this code get called, list frames
bt // thread backtrace

一般主要查找 top_level_code 位置，找到出错未知

// what is the failure conditions
f 1 // frame select 1

p foo // expression foo

b filename.swift:8
b func_name

br s -r timestwo.*String // breakpoint set --func-regex timestwo.*String

b *
br m -c “valueInCents < Int(amountInCents)”  // breakpoint modify --condition “...”
br m -c "...." 3.1


b *
br co a //  breakpoint command add // enter commands, `DONE` to end


br l // breakpoint list

br dis 1 // breakpoint disable 1

### Infos 

The stop reason tells you what happened

- ``EXC_BAD_INSTRUCTION``  Assertion
- ``SIGABRT`` Exception (usually Objective-C) 
- ``EXC_BAD_ACCESS``  Memory error

The stack tells you how it happened
The expression command helps you inspect variables

## swift-demangle

```
xcrun swift-demangle __TIFSs4dumpU__FTQ_4nameGSqSS_6indentSi8maxDepthSi8maxItemsSi_Q_A1_
_TIFSs4dumpU__FTQ_4nameGSqSS_6indentSi8maxDepthSi8maxItemsSi_Q_A1_ --->
Swift.(dump <A>(A, name : Swift.String?, indent : Swift.Int, maxDepth : Swift.Int, maxItems : Swift.Int) -> A).(default argument 2)
```


## dispatch queue

```
import Foundation
var queue : dispatch_queue_t = dispatch_queue_create("my_queue", nil)
dispatch_sync(queue) {println(“world")}
```

## Upcast vs Downcast

A matter of idom

Upcast: T to AnyObject

Downcast: AnyObject ``as`` T

## Swift Array ``T[]``

Two representations.

- Native Array => |len|cap|0|1|2|..|
- Cocoa Array => NSArray obj

## Unmanaged

Used in CF.

```
struct Unmanaged<T: AnyObject> { 
  func takeUnretainedValue() -> T   // for +0 returns
  func takeRetainedValue() -> T   // for +1 returns 

  func retain() -> Unmanaged<T> 
  func release() 
  func autorelease() -> Unmanaged<T>
} 
```

## Weak Reference

Weak references are optional values

- Use strong references from owners to the objects they own
- Use weak references among objects with independent lifetimes
- Use unowned references from owned objects with the same lifetime //  互相绑定的引用，生命周期相同

## Initializer

- Initialize all values before you use them
- Set all stored properties first, then call super.init
- Designated initializers only delegate up
- Convenience initializers only delegate across

## Capture List

避免循环引用

```
init() {
  onChange = {[unowned self] temp in
    self.currentTemp = temp
  }
}
```

## QuickLookObject

```
enum QuickLookObject {
  case Text(String)
  case Int(Int64)
  case UInt(UInt64)
  case Float(Double)
  case Image(Any)
  case Sound(Any)
  case Color(Any)
  case BezierPath(Any)
  case AttributedString(Any)
  case Rectangle(Double, Double, Double, Double)
  case Point(Double, Double)
  case Size(Double, Double)
  case Logical(Bool)
  case Range(UInt64, UInt64)
  case View(Any)
  case Sprite(Any)
  case URL(String)
  case _Raw(UInt8[], String)
}
```

Add Quick Look support to NSObject subclasses only
Implement the debugQuickLookObject() method
```
func debugQuickLookObject() -> AnyObject? {
    return "Some Quick Look type"
}
```
