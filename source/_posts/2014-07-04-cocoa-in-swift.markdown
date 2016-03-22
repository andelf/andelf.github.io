---
layout: post
title: "Cocoa Extensions in Swift ( Cocoa 在 Swift 中所添加的扩展）"
date: 2014-07-04 01:05:55 +0800
comments: true
categories: swift swift-internals
---

最近看到了 [Swift Style Guide](https://github.com/raywenderlich/swift-style-guide) 个人觉得内容太少，
Swift 本身作为一门庞大的语言，语素众多。本文就 Swift 本身对 Cocoa 的扩展，看看对日常 Cocoa 风格有什么影响。

Swift 本身的特性，导致它在一些用法上和 Objective-C 上有所不同，比如 ObjC 的 struct 单纯和 C 的一样，但是在 Swift 
中的 struct 则要强大得多。

个人认为比如 ``CGPointMake`` 这样的函数，理论上不应该出现在 Swift 代码中。而是应该用 ``CGPoint(x:y:)``。

本文可以作为参考手册使用。

## 标准库扩展

### ObjectiveC

值得注意的是 Selector 相关方法，实现了 ``StringLiteralConvertible``。也可以从 ``nil`` 获得。

### Foundation

这里忽略之前介绍过的 ``_BridgedToObjectiveC`` 相关内容。

#### 协议附加

Sequence 协议

```
NSMutableArray NSSet NSArray NSMutableDictionary NSMutableSet NSDictionary
```

所有以上这些类型都可以通过 for-in 操作。

*LiteralConvertible

```
NSNumber NSString NSArray NSDictionary
```

#### 隐式类型转换

CF 几乎都对应到了 NS 类型。这里略去

- ``NilType`` -> ``NSZone``
- ``Dictionary<KeyType: Hashable, ValueType>`` -> ``NSDictionary``
- ``NSDictionary`` -> ``Dictionary<NSObject, AnyObject>``
- ``String`` <-> ``NSString``
- ``NSArray`` -> ``AnyObject[]``
- ``A[]`` -> ``NSArray``
- ``Float Double Int UInt Bool`` -> ``NSNumber``
- ``NSRange`` -> ``Range<Int>`` // 比较有意思的一个 

#### 方法扩展

```scala
// let s = NSSet(objects: 12, 32, 23, 12)
extension NSSet {
  convenience init(objects elements: AnyObject...)
}
extension NSOrderedSet {
  convenience init(objects elements: AnyObject...)
}
// 这里注意，NSRange 和 Swift Range 对 range 结束的表述方法不同
// NSRange 保存 range 元素个数
// Swift Range 保存的是结束元素
// let r = NSRange(0..20)
extension NSRange {
  init(_ x: Range<Int>)
}
// let prop = NSDictionary(objectsAndKeys: "Feather", "name", "Programming", "hobby")
extension NSDictionary {
  convenience init(objectsAndKeys objects: AnyObject...)
}
extension NSObject : CVarArg {
  @objc func encode() -> Word[]
}
```

字符串的扩展方法非常多。

```scala
  static func availableStringEncodings() -> NSStringEncoding[]
  static func defaultCStringEncoding() -> NSStringEncoding
  static func localizedNameOfStringEncoding(encoding: NSStringEncoding) -> String
  static func localizedStringWithFormat(format: String, _ arguments: CVarArg...) -> String
  static func pathWithComponents(components: String[]) -> String
  static func stringWithContentsOfFile(path: String, encoding enc: NSStringEncoding, error: NSErrorPointer = default) -> String?
  static func stringWithContentsOfFile(path: String, usedEncoding: CMutablePointer<NSStringEncoding> = default, error: NSErrorPointer = default) -> String?
  static func stringWithContentsOfURL(url: NSURL, encoding enc: NSStringEncoding, error: NSErrorPointer = default) -> String?
  static func stringWithContentsOfURL(url: NSURL, usedEncoding enc: CMutablePointer<NSStringEncoding> = default, error: NSErrorPointer = default) -> String?
  static func stringWithCString(cString: CString, encoding enc: NSStringEncoding) -> String?
  static func stringWithUTF8String(bytes: CString) -> String?
  func canBeConvertedToEncoding(encoding: NSStringEncoding) -> Bool
  var capitalizedString: String { get }
  func capitalizedStringWithLocale(locale: NSLocale) -> String
  func caseInsensitiveCompare(aString: String) -> NSComparisonResult
  func commonPrefixWithString(aString: String, options: NSStringCompareOptions) -> String
  func compare(aString: String, options mask: NSStringCompareOptions = default, range: Range<String.Index>? = default, locale: NSLocale? = default) -> NSComparisonResult
  func completePathIntoString(_ outputName: CMutablePointer<String> = default, caseSensitive: Bool, matchesIntoArray: CMutablePointer<String[]> = default, filterTypes: String[]? = default) -> Int
  func componentsSeparatedByCharactersInSet(separator: NSCharacterSet) -> String[]
  func componentsSeparatedByString(separator: String) -> String[]
  func cStringUsingEncoding(encoding: NSStringEncoding) -> CChar[]?
  func dataUsingEncoding(encoding: NSStringEncoding, allowLossyConversion: Bool = default) -> NSData
  var decomposedStringWithCanonicalMapping: String { get }
  var decomposedStringWithCompatibilityMapping: String { get }
  func enumerateLines(body: (line: String, inout stop: Bool) -> ())
  func enumerateLinguisticTagsInRange(range: Range<String.Index>, scheme tagScheme: String, options opts: NSLinguisticTaggerOptions, orthography: NSOrthography?, _ body: (String, Range<String.Index>, Range<String.Index>, inout Bool) -> ())
  func enumerateSubstringsInRange(range: Range<String.Index>, options opts: NSStringEnumerationOptions, _ body: (substring: String, substringRange: Range<String.Index>, enclosingRange: Range<String.Index>, inout Bool) -> ())
  var fastestEncoding: NSStringEncoding { get }
  func fileSystemRepresentation() -> CChar[]
  func getBytes(inout buffer: UInt8[], maxLength: Int, usedLength: CMutablePointer<Int>, encoding: NSStringEncoding, options: NSStringEncodingConversionOptions, range: Range<String.Index>, remainingRange: CMutablePointer<Range<String.Index>>) -> Bool
  func getCString(inout buffer: CChar[], maxLength: Int, encoding: NSStringEncoding) -> Bool
  func getFileSystemRepresentation(inout buffer: CChar[], maxLength: Int) -> Bool
  func getLineStart(start: CMutablePointer<String.Index>, end: CMutablePointer<String.Index>, contentsEnd: CMutablePointer<String.Index>, forRange: Range<String.Index>)
  func getParagraphStart(start: CMutablePointer<String.Index>, end: CMutablePointer<String.Index>, contentsEnd: CMutablePointer<String.Index>, forRange: Range<String.Index>)
  var hash: Int { get }
  static func stringWithBytes(bytes: UInt8[], length: Int, encoding: NSStringEncoding) -> String?
  static func stringWithBytesNoCopy(bytes: CMutableVoidPointer, length: Int, encoding: NSStringEncoding, freeWhenDone flag: Bool) -> String?
  init(utf16CodeUnits: CConstPointer<unichar>, count: Int)
  init(utf16CodeUnitsNoCopy: CConstPointer<unichar>, count: Int, freeWhenDone flag: Bool)
  init(format: String, _ _arguments: CVarArg...)
  init(format: String, arguments: CVarArg[])
  init(format: String, locale: NSLocale?, _ args: CVarArg...)
  init(format: String, locale: NSLocale?, arguments: CVarArg[])
  var lastPathComponent: String { get }
  var utf16count: Int { get }
  func lengthOfBytesUsingEncoding(encoding: NSStringEncoding) -> Int
  func lineRangeForRange(aRange: Range<String.Index>) -> Range<String.Index>
  func linguisticTagsInRange(range: Range<String.Index>, scheme tagScheme: String, options opts: NSLinguisticTaggerOptions = default, orthography: NSOrthography? = default, tokenRanges: CMutablePointer<Range<String.Index>[]> = default) -> String[]
  func localizedCaseInsensitiveCompare(aString: String) -> NSComparisonResult
  func localizedCompare(aString: String) -> NSComparisonResult
  func localizedStandardCompare(string: String) -> NSComparisonResult
  func lowercaseStringWithLocale(locale: NSLocale) -> String
  func maximumLengthOfBytesUsingEncoding(encoding: NSStringEncoding) -> Int
  func paragraphRangeForRange(aRange: Range<String.Index>) -> Range<String.Index>
  var pathComponents: String[] { get }
  var pathExtension: String { get }
  var precomposedStringWithCanonicalMapping: String { get }
  var precomposedStringWithCompatibilityMapping: String { get }
  func propertyList() -> AnyObject
  func propertyListFromStringsFileFormat() -> Dictionary<String, String>
  func rangeOfCharacterFromSet(aSet: NSCharacterSet, options mask: NSStringCompareOptions = default, range aRange: Range<String.Index>? = default) -> Range<String.Index>
  func rangeOfComposedCharacterSequenceAtIndex(anIndex: String.Index) -> Range<String.Index>
  func rangeOfComposedCharacterSequencesForRange(range: Range<String.Index>) -> Range<String.Index>
  func rangeOfString(aString: String, options mask: NSStringCompareOptions = default, range searchRange: Range<String.Index>? = default, locale: NSLocale? = default) -> Range<String.Index>
  var smallestEncoding: NSStringEncoding { get }
  func stringByAbbreviatingWithTildeInPath() -> String
  func stringByAddingPercentEncodingWithAllowedCharacters(allowedCharacters: NSCharacterSet) -> String
  func stringByAddingPercentEscapesUsingEncoding(encoding: NSStringEncoding) -> String
  func stringByAppendingFormat(format: String, _ arguments: CVarArg...) -> String
  func stringByAppendingPathComponent(aString: String) -> String
  func stringByAppendingPathExtension(ext: String) -> String
  func stringByAppendingString(aString: String) -> String
  var stringByDeletingLastPathComponent: String { get }
  var stringByDeletingPathExtension: String { get }
  var stringByExpandingTildeInPath: String { get }
  func stringByFoldingWithOptions(options: NSStringCompareOptions, locale: NSLocale) -> String
  func stringByPaddingToLength(newLength: Int, withString padString: String, startingAtIndex padIndex: Int) -> String
  var stringByRemovingPercentEncoding: String { get }
  func stringByReplacingCharactersInRange(range: Range<String.Index>, withString replacement: String) -> String
  func stringByReplacingOccurrencesOfString(target: String, withString replacement: String, options: NSStringCompareOptions = default, range searchRange: Range<String.Index>? = default) -> String
  func stringByReplacingPercentEscapesUsingEncoding(encoding: NSStringEncoding) -> String
  var stringByResolvingSymlinksInPath: String { get }
  var stringByStandardizingPath: String { get }
  func stringByTrimmingCharactersInSet(set: NSCharacterSet) -> String
  func stringsByAppendingPaths(paths: String[]) -> String[]
  func substringFromIndex(index: Int) -> String
  func substringToIndex(index: Int) -> String
  func substringWithRange(aRange: Range<String.Index>) -> String
  func uppercaseStringWithLocale(locale: NSLocale) -> String
  func writeToFile(path: String, atomically useAuxiliaryFile: Bool, encoding enc: NSStringEncoding, error: NSErrorPointer = default) -> Bool
  func writeToURL(url: NSURL, atomically useAuxiliaryFile: Bool, encoding enc: NSStringEncoding, error: NSErrorPointer = default) -> Bool
```

### CoreGraphics 

几个常用基本类型都有了 Swift-style 的构造函数。其中 ``CGRect`` 有很多的相关运算都被封装为方法，很不错。

```scala
extension CGPoint : Equatable {
  static var zeroPoint: CGPoint
  init()
  init(x: Int, y: Int)
}
extension CGSize {
  static var zeroSize: CGSize
  init()
  init(width: Int, height: Int)
}
extension CGVector {
  static var zeroVector: CGVector
  init(_ dx: CGFloat, _ dy: CGFloat)
  init(_ dx: Int, _ dy: Int)
}
extension CGRect : Equatable {
  // 全为 0
  static var zeroRect: CGRect
  // 原点为无穷大，表示空
  static var nullRect: CGRect
  // 原点无穷小，宽高无穷大
  static var infiniteRect: CGRect
  init()
  init(x: CGFloat, y: CGFloat, width: CGFloat, height: CGFloat)
  init(x: Int, y: Int, width: Int, height: Int)
  var width: CGFloat
  var height: CGFloat
  var minX: CGFloat
  var minY: CGFloat
  // 中点
  var midX: CGFloat
  var midY: CGFloat
  var maxX: CGFloat
  var maxY: CGFloat
  var isNull: Bool
  var isEmpty: Bool
  var isInfinite: Bool
  var standardizedRect: CGRect
  func standardize()
  var integerRect: CGRect
  func integerize()
  func rectByInsetting(#dx: CGFloat, dy: CGFloat) -> CGRect
  func inset(#dx: CGFloat, dy: CGFloat)
  func rectByOffsetting(#dx: CGFloat, dy: CGFloat) -> CGRect
  func offset(#dx: CGFloat, dy: CGFloat)
  func rectByUnion(withRect: CGRect) -> CGRect
  func union(withRect: CGRect)
  func rectByIntersecting(withRect: CGRect) -> CGRect
  func intersect(withRect: CGRect)
  func rectsByDividing(atDistance: CGFloat, fromEdge: CGRectEdge) -> (slice: CGRect, remainder: CGRect)
  func contains(rect: CGRect) -> Bool
  func contains(point: CGPoint) -> Bool
  func intersects(rect: CGRect) -> Bool
}
```

### AppKit

```scala
extension NSGradient {
  convenience init(colorsAndLocations objects: (AnyObject, CGFloat)...)
}
```

### UIKit

```scala
extension UIDeviceOrientation {
  var isPortrait: Bool
  // also isLandscape isValidInterfaceOrientation isFlat 
}
extension UIInterfaceOrientation {
  var isPortrait: Bool
  var isLandscape: Bool
}
```

这个模块是交叉编译的。。不太容易获得信息。不过好在扩展内容不多。

### SpriteKit

```scala
extension SKNode {
  @objc subscript (name: String) -> SKNode[] { get }
}
```

## 特殊 Mirror 实现

```
NSSet NSDate NSArray NSRange NSURL NSDictionary NSString
CGPoint CGRect CGSize
NSView
UIView
SKTextureAtlas SKTexture SKSpriteNode SKShapeNode
```

单独添加了自己的 ``Mirror`` 类型，单独实现。

``Mirror`` 类型其实是为 ``QuickLookObject`` 准备的，也就是在 Xcode Playground 中快速查看。

