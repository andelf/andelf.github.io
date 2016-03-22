---
layout: post
title: "Swift Undocumented Grammar （Swift 黑语法）"
date: 2014-07-04 03:05:11 +0800
comments: true
categories: swift swift-internals
---

本文介绍 Swift 的 Undocumented 语法特性。

电子书上介绍的 default function parameter 这里都不好意思拿出来写。

咳咳。持续更新。

## 用关键字当变量名

Keywards as variable name.

```scala
// escaped variable name
let `let` = 1000
dump(`let`, name: "variable named let")
```

## ``new`` 关键字

The ``new`` keyword.

快速初始化数组。

```scala
let an_array_with_100_zero = new(Int)[100]
```

## protocol type

use ``protocol<Protocol1, Protocol2, ...>`` as a type.

## How I find it?

瞎试出来的。

