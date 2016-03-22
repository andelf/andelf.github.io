---
layout: post
title: "Swift Runtime"
date: 2014-06-16 10:38:45 +0800
comments: true
categories: swift
---


Swift is written in C++ 、Objective-C、Swift、 Assembly.

TO BE CONTINUED

libdyld.dylib`start

```c
main(argc, argv) {
  once() { C_ARGC = argc }
  once() { C_ARGV = argv }
  top_level_code()
}
```

All code in swift file goes to ``top_level_code``。
