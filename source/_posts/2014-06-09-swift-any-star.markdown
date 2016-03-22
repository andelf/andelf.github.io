---
layout: post
title: "Swift Any*"
date: 2014-06-09 00:40:52 +0800
comments: true
categories: swift
---

```scala
typealias Any = protocol<>

typealias AnyClass = AnyObject.Type

@class_protocol protocol AnyObject {
}

// Also there is another type:
Any.Type

```
Metatype Type

“The metatype of a class, structure, or enumeration type is the name of that type followed by .Type. The metatype of a protocol type—not the concrete type that conforms to the protocol at runtime—is the name of that protocol followed by .Protocol. For example, the metatype of the class type SomeClass is SomeClass.Type and the metatype of the protocol SomeProtocol is SomeProtocol.Protocol.”

摘录来自: Apple Inc. “The Swift Programming Language”。 iBooks. https://itun.es/cn/jEUH0.l

“You can use the postfix self expression to access a type as a value. For example, SomeClass.self returns SomeClass itself, not an instance of SomeClass. And SomeProtocol.self returns SomeProtocol itself, not an instance of a type that conforms to SomeProtocol at runtime. You can use a dynamicType expression with an instance of a type to access that instance’s runtime type as a value”

摘录来自: Apple Inc. “The Swift Programming Language”。 iBooks. https://itun.es/cn/jEUH0.l

