---
layout: post
title: "Swift 2.0 的错误处理(Swift 2.0 Error Handling)"
date: 2015-06-09 15:27:58 +0800
comments: true
categories: blog
tags:
- swift
published: false
---

```
protocol ErrorType {
  var _domain: String { get }
  var _code: Int { get }
}

@asmname("swift_bridgeErrorTypeToNSError") func _bridgeErrorTypeToNSError(e: ErrorType) -> AnyObject

@asmname("swift_stdlib_getErrorCode") func _stdlib_getErrorCode<T : ErrorType>(x: UnsafePointer<T>) -> Int

@asmname("swift_stdlib_getErrorDomainNSString") func _stdlib_getErrorDomainNSString<T : ErrorType>(x: UnsafePointer<T>) -> AnyObject
```


## Foundation

```
protocol _ObjectiveCBridgeableErrorType : ErrorType {
  init?(_bridgedNSError: NSError)
}

struct NSCocoaError : RawRepresentable, _BridgedNSError, _ObjectiveCBridgeableErrorType, ErrorType, __BridgedNSError, Hashable, Equatable {
  let rawValue: Int
  init(rawValue: Int)
  static var _NSErrorDomain: String {
    get {}
  }
  typealias RawValue = Int
}


infix func ==(a: _GenericObjCError, b: _GenericObjCError) -> Bool
infix func ==(a: _GenericObjCError, b: _GenericObjCError) -> Bool
func ==<T : __BridgedNSError where T.RawValue : SignedIntegerType>(lhs: T, rhs: T) -> Bool

@available(OSX 10.11, iOS 9.0, *)
func resolveError(error: NSError?) throws

enum _GenericObjCError : ErrorType {
  case NilError
  var hashValue: Int {
    get {}
  }
  var _domain: String {
    get {}
  }
  var _code: Int {
    get {}
  }
}

@asmname("swift_stdlib_bridgeNSErrorToErrorType")
func _stdlib_bridgeNSErrorToErrorType<T : _ObjectiveCBridgeableErrorType>(error: NSError, out: UnsafeMutablePointer<T>) -> Bool

@asmname("swift_convertNSErrorToErrorType") func _convertNSErrorToErrorType(error: NSError?) -> ErrorType

@objc enum NSURLError : Int, _BridgedNSError, _ObjectiveCBridgeableErrorType, ErrorType, __BridgedNSError { ... }


protocol __BridgedNSError : RawRepresentable {
  static var _NSErrorDomain: String { get }
}
@asmname("swift_convertErrorTypeToNSError") func _convertErrorTypeToNSError(error: ErrorType) -> NSError
func ~=(match: NSCocoaError, error: ErrorType) -> Bool
protocol _BridgedNSError : __BridgedNSError, _ObjectiveCBridgeableErrorType, Hashable {
  static var _NSErrorDomain: String { get }
}
```

ErrorType 在 Swift 中表示。

```
extension NSError : ErrorType {
  @objc dynamic var _domain: String {
    @objc dynamic get {}
  }
  @objc dynamic var _code: Int {
    @objc dynamic get {}
  }
}
```
