---
layout: post
title: "Swift Special Protocols"
date: 2014-06-13 12:43:07 +0800
comments: true
categories: swift
---

via WWDC 404 

Protocols are your hooks into the Swift core language
Swift generics combine abstraction, safety, and performance in new ways
Read, experiment, and have fun. There’s plenty to discover!

## ``LogicValue`` // 重载 if val

## ``Printable`` // 重载 println
```scala
protocol Printable {
    var description: String { get }
}
```

## ``Sequence`` // 重载 for-in

```scala
protocol Sequence {
  typealias GeneratorType : Generator
  func generate() -> GeneratorType
}

protocol Generator {
  typealias Element
  mutating func next() -> Element?
}
```

```scala
// for .. in { }
var __g = someSequence.generate()
while let x = __g.next() {
    ...
}}
```
## *Convertible

- IntegerLiteralConvertible
- FloatLiteralConvertible
- StringLiteralConvertible
- ArrayLiteralConvertible
- DictionaryLiteralConvertible

## Equatable

``==``

