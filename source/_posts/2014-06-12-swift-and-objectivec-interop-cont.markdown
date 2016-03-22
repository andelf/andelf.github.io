---
layout: post
title: "Swift and ObjectiveC Interop Cont. (Swift 与 Objective-C 之间的交互，续坑)"
date: 2014-06-12 17:10:28 +0800
comments: true
categories: swift swift-interop
---

## 关于 ``_ConditionallyBridgedToObjectiveC``

When you bridge from a Swift array to an NSArray object, the elements in the Swift array must be AnyObject compatible. For example, a Swift array of type Int[] contains Int structure elements. The Int type is not an instance of a class, but because the Int type bridges to the NSNumber class, the Int type is AnyObject compatible. Therefore, you can bridge a Swift array of type Int[] to an NSArray object. If an element in a Swift array is not AnyObject compatible, a runtime error occurs when you bridge to an NSArray object.

## 其他 Tips

When you declare an outlet in Swift, the compiler automatically converts the type to a weak implicitly unwrapped optional and assigns it an initial value of nil. In effect, the compiler replaces @IBOutlet var name: Type with @IBOutlet weak var name: Type! = nil. 

In Swift, there are no readwrite and readonly attributes. When declaring a stored property, use let to make it read-only, and use var to make it read/write. When declaring a computed property, provide a getter only to make it read-only and provide both a getter and setter to make it read/write.

