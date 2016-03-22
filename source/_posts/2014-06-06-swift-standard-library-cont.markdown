---
layout: post
title: "Swift Standard Library Cont. (Swift 标准库 第二部分)"
date: 2014-06-06 13:33:52 +0800
comments: true
categories: swift
---

UPDATE: 我还真没用过 Xcode。
输入 import Swift，按住 Command 鼠标点击 Swift。然后就出来了。 ORZ。 via reddit。

不过那个列表是不全的，有部分影藏的还是要慢慢来。比如 NilType.

通过命令行 lldb 命令获得相关信息。转载注明。

所有默认名字在 Swift 名字空间，即 Swift.abs 这样。这里是第二部分，小写符号请参考，前一篇。

## 第二部分 A to Z

### ``AbsoluteValuable``

```scala
protocol AbsoluteValuable : SignedNumber {
  class func abs(_: Self) -> Self
}
```

### ``Any``
```scala
typealias Any = protocol<>
```

### ``AnyClass``
```scala
typealias AnyClass = AnyObject.Type
```

### ``AnyObject``
```scala
@objc @class_protocol protocol AnyObject {}
```

### ``Array``
```scala
struct Array<T> : MutableCollection, Sliceable {
  typealias Element = T
  var startIndex: Int {
    get {}
  }
  var endIndex: Int {
    get {}
  }
  subscript (index: Int) -> T {
    get {}
    set {}
  }
  func generate() -> IndexingGenerator<Array<T>>
  typealias SliceType = Slice<T>
  subscript (subRange: Range<Int>) -> Slice<T> {
    get {}
    set(rhs) {}
  }
  typealias _Buffer = ArrayBuffer<T>
  init(_ buffer: ArrayBuffer<T>)
  var _buffer: ArrayBuffer<T>
}
extension Array<T> : ArrayLiteralConvertible {
  static func convertFromArrayLiteral(elements: T...) -> Array<T>
}
extension Array<T> {
  func _asCocoaArray() -> _CocoaArray
}
extension Array<T> : ArrayType {
  init()
  init<S : Sequence where T == T>(_ s: S)
  init(count: Int, repeatedValue: T)
  var count: Int {
    get {}
  }
  var capacity: Int {
    get {}
  }
  var isEmpty: Bool {
    get {}
  }
  var _owner: AnyObject? {
    get {}
  }
  var _elementStorageIfContiguous: UnsafePointer<T> {
    get {}
  }
  func copy() -> Array<T>
  func unshare()
  func reserveCapacity(minimumCapacity: Int)
  func append(newElement: T)
  func extend<S : Sequence where T == T>(sequence: S)
  func removeLast() -> T
  func insert(newElement: T, atIndex: Int)
  func removeAtIndex(index: Int) -> T
  func removeAll(keepCapacity: Bool = default)
  func join<S : Sequence where Array<T> == Array<T>>(elements: S) -> Array<T>
  func reduce<U>(initial: U, combine: (U, T) -> U) -> U
  func sort(isOrderedBefore: (T, T) -> Bool)
  func map<U>(transform: (T) -> U) -> Array<U>
  func reverse() -> Array<T>
  func filter(includeElement: (T) -> Bool) -> Array<T>
}
extension Array<T> : Reflectable {
  func getMirror() -> Mirror
}
extension Array<T> : Printable, DebugPrintable {
  func _makeDescription(#isDebug: Bool) -> String
  var description: String {
    get {}
  }
  var debugDescription: String {
    get {}
  }
}
extension Array<T> {
  @transparent func _cPointerArgs() -> (AnyObject?, RawPointer)
  @conversion @transparent func __conversion() -> CConstPointer<T>
  @conversion @transparent func __conversion() -> CConstVoidPointer
}
extension Array<T> {
  func withUnsafePointerToElements<R>(body: (UnsafePointer<T>) -> R) -> R
}
extension Array<T> {
  static func convertFromHeapArray(base: RawPointer, owner: NativeObject, count: Word) -> Array<T>
}
extension Array<T> {
  func replaceRange<C : Collection where T == T>(subRange: Range<Int>, with newValues: C)
}
```

### ``ArrayBound``
```scala
protocol ArrayBound {
  typealias ArrayBoundType
  func getArrayBoundValue() -> ArrayBoundType
}
```

### ``ArrayBuffer``
```scala
struct ArrayBuffer<T> : ArrayBufferType {
  var storage: NativeObject?
  var indirect: IndirectArrayBuffer {
    get {}
  }
  typealias Element = T
  init()
  init(_ cocoa: _CocoaArray)
  init(_ buffer: IndirectArrayBuffer)
  func castToBufferOf<U>(_: U.Type) -> ArrayBuffer<U>
}
extension ArrayBuffer<T> {
  init(_ source: ContiguousArrayBuffer<T>)
  func isUniquelyReferenced() -> Bool
  func _asCocoaArray() -> _CocoaArray
  var _hasMutableBuffer: Bool {
    get {}
  }
  func requestUniqueMutableBuffer(minimumCapacity: Int) -> ContiguousArrayBuffer<T>?
  func requestNativeBuffer() -> ContiguousArrayBuffer<T>?
  func _typeCheck(subRange: Range<Int>)
  func _uninitializedCopy(subRange: Range<Int>, target: UnsafePointer<T>) -> UnsafePointer<T>
  subscript (subRange: Range<Int>) -> SliceBuffer<T> {
    get {}
  }
  var elementStorage: UnsafePointer<T> {
    get {}
  }
  var count: Int {
    get {}
    set {}
  }
  var capacity: Int {
    get {}
  }
  subscript (i: Int) -> T {
    get {}
    set {}
  }
  func withUnsafePointerToElements<R>(body: (UnsafePointer<T>) -> R) -> R
  var owner: AnyObject? {
    get {}
  }
  var identity: Word {
    get {}
  }
  var startIndex: Int {
    get {}
  }
  var endIndex: Int {
    get {}
  }
  func generate() -> IndexingGenerator<ArrayBuffer<T>>
  typealias Storage = ContiguousArrayStorage<T>
  typealias NativeBuffer = ContiguousArrayBuffer<T>
  func _invariantCheck() -> Bool
  var _isNative: Bool {
    get {}
  }
  var _native: ContiguousArrayBuffer<T> {
    get {}
  }
  var _nonNative: _CocoaArray? {
    get {}
  }
}
```

### ``ArrayBufferType``
```scala
protocol ArrayBufferType : MutableCollection {
  typealias Element
  init()
  init(_ buffer: ContiguousArrayBuffer<Element>)
  func _uninitializedCopy(subRange: Range<Int>, target: UnsafePointer<Element>) -> UnsafePointer<Element>
  func _asCocoaArray() -> _CocoaArray
  subscript (index: Int) -> Element { get set }
  func requestUniqueMutableBuffer(minimumCapacity: Int) -> ContiguousArrayBuffer<Element>?
  func requestNativeBuffer() -> ContiguousArrayBuffer<Element>?
  subscript (subRange: Range<Int>) -> SliceBuffer<Element> { get }
  func withUnsafePointerToElements<R>(body: (UnsafePointer<Element>) -> R) -> R
  var count: Int { get set }
  var capacity: Int { get }
  var owner: AnyObject? { get }
  var elementStorage: UnsafePointer<Element> { get }
  var identity: Word { get }
}
```


### ``ArrayLiteralConvertible``
```scala
protocol ArrayLiteralConvertible {
  typealias Element
  class func convertFromArrayLiteral(elements: Element...) -> Self
}
```

### ``ArrayType``
```scala
protocol ArrayType : _ArrayType, ExtensibleCollection, MutableSliceable, ArrayLiteralConvertible {
  init()
  init(count: Int, repeatedValue: Self.GeneratorType.Element)
  var count: Int { get }
  var capacity: Int { get }
  var isEmpty: Bool { get }
  var _owner: AnyObject? { get }
  var _elementStorageIfContiguous: UnsafePointer<Self.Element> { get }
  subscript (index: Int) -> Self.GeneratorType.Element { get set }
  func reserveCapacity(minimumCapacity: Int)
  func append(newElement: Self.GeneratorType.Element)
  func extend<S : Sequence where `Self`.GeneratorType.Element == Self.GeneratorType.Element>(sequence: S)
  func removeLast() -> Self.GeneratorType.Element
  func insert(newElement: Self.GeneratorType.Element, atIndex: Int)
  func removeAtIndex(index: Int) -> Self.GeneratorType.Element
  func removeAll(#keepCapacity: Bool)
  func join<S : Sequence where `Self` == Self>(elements: S) -> Self
  func reduce<U>(initial: U, combine: (U, Self.GeneratorType.Element) -> U) -> U
  func sort(isOrderedBefore: (Self.GeneratorType.Element, Self.GeneratorType.Element) -> Bool)
  typealias _Buffer : ArrayBufferType
  init(_ buffer: _Buffer)
  var _buffer: _Buffer { get set }
}
```

```scala
protocol _ArrayType : Collection {
  var count: Int { get }
  typealias _Buffer : ArrayBufferType
  var _buffer: _Buffer { get }
}
```

### ``AutoreleasingUnsafePointer``
```scala
struct AutoreleasingUnsafePointer<T> : Equatable, LogicValue {
  let value: RawPointer
  @transparent init(_ value: RawPointer)
  @transparent static func __writeback_conversion_get(x: T) -> RawPointer
  @transparent static func __writeback_conversion_set(x: RawPointer) -> T
  @transparent static func __writeback_conversion(inout autoreleasingTemp: RawPointer) -> AutoreleasingUnsafePointer<T>
  var _isNull: Bool {
    @transparent get {}
  }
  @transparent func getLogicValue() -> Bool
  var memory: T {
    @transparent get {}
    @transparent set {}
  }
}
```

### ``BidirectionalIndex``
```scala
protocol BidirectionalIndex : ForwardIndex, _BidirectionalIndex {}
```

```scala
protocol _BidirectionalIndex : _ForwardIndex {
  func pred() -> Self
}
```

### ``Bit``
```scala
enum Bit : Int, RandomAccessIndex {
  case zero
  case one
  func succ() -> Bit
  func pred() -> Bit
  func distanceTo(other: Bit) -> Int
  func advancedBy(distance: Int) -> Bit
  var hashValue: Int {
    get {}
  }
  static func fromRaw(raw: Int) -> Bit?
  func toRaw() -> Int
}
extension Bit : IntegerArithmetic {
  static func _withOverflow(x: Int, _ b: Bool) -> (Bit, Bool)
  static func uncheckedAdd(lhs: Bit, _ rhs: Bit) -> (Bit, Bool)
  static func uncheckedSubtract(lhs: Bit, _ rhs: Bit) -> (Bit, Bool)
  static func uncheckedMultiply(lhs: Bit, _ rhs: Bit) -> (Bit, Bool)
  static func uncheckedDivide(lhs: Bit, _ rhs: Bit) -> (Bit, Bool)
  static func uncheckedModulus(lhs: Bit, _ rhs: Bit) -> (Bit, Bool)
  func toIntMax() -> IntMax
}
```

### ``BitwiseOperations``
```scala
protocol BitwiseOperations {
  func &(_: Self, _: Self) -> Self
  func |(_: Self, _: Self) -> Self
  func ^(_: Self, _: Self) -> Self
  func ~(_: Self) -> Self
  class var allZeros: Self { get }
}
```

### ``Bool``
```scala
struct Bool {
  var value: Int1
  @transparent init()
  @transparent init(_ v: Int1)
  static var false: Bool {
    @transparent get {}
  }
  static var true: Bool {
    @transparent get {}
  }
}
extension Bool : LogicValue {
  @transparent func _getBuiltinLogicValue() -> Int1
  @transparent func getLogicValue() -> Bool
  init(_ v: LogicValue)
}
extension Bool : Printable {
  var description: String {
    get {}
  }
}
extension Bool : Equatable, Hashable {
  var hashValue: Int {
    @transparent get {}
  }
}
extension Bool : Reflectable {
  func getMirror() -> Mirror
}
```

### ``CBool``
```scala
typealias CBool = Bool
```

### ``CChar``
```scala
typealias CChar = Int8
```

### ``CChar16``
```scala
typealias CChar16 = UInt16
```

### ``CChar32``
```scala
typealias CChar32 = UnicodeScalar
```

### ``CConstPointer``
```scala
struct CConstPointer<T> : Equatable {
  let owner: AnyObject?
  let value: RawPointer
  @transparent init(_ owner: AnyObject?, _ value: RawPointer)
  @transparent static func __inout_conversion(inout scalar: T) -> CConstPointer<T>
  var scoped: Bool {
    @transparent get {}
  }
  @transparent func withUnsafePointer<U>(f: UnsafePointer<T> -> U) -> U
}
```

### ``CConstVoidPointer``
```scala
struct CConstVoidPointer : Equatable {
  let owner: AnyObject?
  let value: RawPointer
  @transparent init(_ owner: AnyObject?, _ value: RawPointer)
  @transparent static func __inout_conversion<T>(inout scalar: T) -> CConstVoidPointer
  var scoped: Bool {
    @transparent get {}
  }
  @transparent func withUnsafePointer<T, U>(f: UnsafePointer<T> -> U) -> U
}
```

### ``CDouble``
```scala
typealias CDouble = Double
```

### ``CFloat``
```scala
typealias CFloat = Float
```

### ``Character``
```scala
enum Character : _BuiltinExtendedGraphemeClusterLiteralConvertible, ExtendedGraphemeClusterLiteralConvertible, Equatable {
  case LargeRepresentation(OnHeap<String>)
  case SmallRepresentation(Int63)
  init(_ scalar: UnicodeScalar)
  static func _convertFromBuiltinExtendedGraphemeClusterLiteral(start: RawPointer, byteSize: Word, isASCII: Int1) -> Character
  static func convertFromExtendedGraphemeClusterLiteral(value: Character) -> Character
  init(_ s: String)
  static func _smallSize(value: UInt64) -> Int
  static func _smallValue(value: Int63) -> UInt64
}
extension Character : Streamable {
  func writeTo<Target : OutputStream>(inout target: Target)
}
```

### ``CharacterLiteralConvertible``
```scala
protocol CharacterLiteralConvertible {
  typealias CharacterLiteralType : _BuiltinCharacterLiteralConvertible
  class func convertFromCharacterLiteral(value: CharacterLiteralType) -> Self
}
```

### ``CInt``
```scala
typealias CInt = Int32
```

### ``CLong``
```scala
typealias CLong = Int
```

### ``CLongLong``
```scala
typealias CLongLong = Int64
```

### ``CMutablePointer``
```scala
struct CMutablePointer<T> : Equatable, LogicValue {
  let owner: AnyObject?
  let value: RawPointer
  @transparent static func __inout_conversion(inout scalar: T) -> CMutablePointer<T>
  @transparent static func __inout_conversion(inout a: Array<T>) -> CMutablePointer<T>
  var scoped: Bool {
    @transparent get {}
  }
  @transparent func withUnsafePointer<U>(f: UnsafePointer<T> -> U) -> U
  @transparent func getLogicValue() -> Bool
  @transparent func _withBridgeObject<U, R>(inout buffer: U?, body: (AutoreleasingUnsafePointer<U?>) -> R) -> R
  @transparent func _withBridgeValue<U, R>(inout buffer: U, body: (CMutablePointer<U>) -> R) -> R
  func _setIfNonNil(body: () -> T)
  init(owner: AnyObject?, value: RawPointer)
}
```

### ``CMutableVoidPointer``
```scala
struct CMutableVoidPointer : Equatable {
  let owner: AnyObject?
  let value: RawPointer
  @transparent static func __inout_conversion<T>(inout scalar: T) -> CMutableVoidPointer
  @transparent static func __inout_conversion<T>(inout a: Array<T>) -> CMutableVoidPointer
  var scoped: Bool {
    @transparent get {}
  }
  @transparent func withUnsafePointer<T, U>(f: UnsafePointer<T> -> U) -> U
  init(owner: AnyObject?, value: RawPointer)
}
```

### ``Collection``
```scala
protocol Collection : _Collection, Sequence {
  subscript (i: Self.IndexType) -> Self.GeneratorType.Element { get }
  func ~>(_: Self, _: (_CountElements, ())) -> Self.IndexType.DistanceType
}
```

### ``CollectionOfOne``
```scala
struct CollectionOfOne<T> : Collection {
  typealias IndexType = Bit
  init(_ element: T)
  var startIndex: IndexType {
    get {}
  }
  var endIndex: IndexType {
    get {}
  }
  func generate() -> GeneratorOfOne<T>
  subscript (i: IndexType) -> T {
    get {}
  }
  let element: T
}
```

### ``Comparable``
```scala
protocol Comparable : _Comparable, Equatable {
  func <=(lhs: Self, rhs: Self) -> Bool
  func >=(lhs: Self, rhs: Self) -> Bool
  func >(lhs: Self, rhs: Self) -> Bool
}
```

```scala
protocol _Comparable {
  func <(lhs: Self, rhs: Self) -> Bool
}
```

### ``ContiguousArray``
```scala
struct ContiguousArray<T> : MutableCollection, Sliceable {
  typealias Element = T
  var startIndex: Int {
    get {}
  }
  var endIndex: Int {
    get {}
  }
  subscript (index: Int) -> T {
    get {}
    set {}
  }
  func generate() -> IndexingGenerator<ContiguousArray<T>>
  typealias SliceType = Slice<T>
  subscript (subRange: Range<Int>) -> Slice<T> {
    get {}
    set(rhs) {}
  }
  typealias _Buffer = ContiguousArrayBuffer<T>
  init(_ buffer: ContiguousArrayBuffer<T>)
  var _buffer: ContiguousArrayBuffer<T>
}
extension ContiguousArray<T> : ArrayLiteralConvertible {
  static func convertFromArrayLiteral(elements: T...) -> ContiguousArray<T>
}
extension ContiguousArray<T> {
  func _asCocoaArray() -> _CocoaArray
}
extension ContiguousArray<T> : ArrayType {
  init()
  init<S : Sequence where T == T>(_ s: S)
  init(count: Int, repeatedValue: T)
  var count: Int {
    get {}
  }
  var capacity: Int {
    get {}
  }
  var isEmpty: Bool {
    get {}
  }
  var _owner: AnyObject? {
    get {}
  }
  var _elementStorageIfContiguous: UnsafePointer<T> {
    get {}
  }
  var _elementStorage: UnsafePointer<T> {
    get {}
  }
  func copy() -> ContiguousArray<T>
  func unshare()
  func reserveCapacity(minimumCapacity: Int)
  func append(newElement: T)
  func extend<S : Sequence where T == T>(sequence: S)
  func removeLast() -> T
  func insert(newElement: T, atIndex: Int)
  func removeAtIndex(index: Int) -> T
  func removeAll(keepCapacity: Bool = default)
  func join<S : Sequence where ContiguousArray<T> == ContiguousArray<T>>(elements: S) -> ContiguousArray<T>
  func reduce<U>(initial: U, combine: (U, T) -> U) -> U
  func sort(isOrderedBefore: (T, T) -> Bool)
  func map<U>(transform: (T) -> U) -> ContiguousArray<U>
  func reverse() -> ContiguousArray<T>
  func filter(includeElement: (T) -> Bool) -> ContiguousArray<T>
}
extension ContiguousArray<T> : Reflectable {
  func getMirror() -> Mirror
}
extension ContiguousArray<T> : Printable, DebugPrintable {
  func _makeDescription(#isDebug: Bool) -> String
  var description: String {
    get {}
  }
  var debugDescription: String {
    get {}
  }
}
extension ContiguousArray<T> {
  @transparent func _cPointerArgs() -> (AnyObject?, RawPointer)
  @conversion @transparent func __conversion() -> CConstPointer<T>
  @conversion @transparent func __conversion() -> CConstVoidPointer
}
extension ContiguousArray<T> {
  func withUnsafePointerToElements<R>(body: (UnsafePointer<T>) -> R) -> R
}
extension ContiguousArray<T> {
  func replaceRange<C : Collection where T == T>(subRange: Range<Int>, with newValues: C)
}
```

### ``ContiguousArrayBuffer``
```scala
struct ContiguousArrayBuffer<T> : ArrayBufferType, LogicValue {
  init(count: Int, minimumCapacity: Int)
  init(_ storage: ContiguousArrayStorage<T>?)
  func getLogicValue() -> Bool
  var elementStorage: UnsafePointer<T> {
    get {}
  }
  var _unsafeElementStorage: UnsafePointer<T> {
    get {}
  }
  func withUnsafePointerToElements<R>(body: (UnsafePointer<T>) -> R) -> R
  func take() -> ContiguousArrayBuffer<T>
  typealias Element = T
  init()
  init(_ buffer: ContiguousArrayBuffer<T>)
  func requestUniqueMutableBuffer(minimumCapacity: Int) -> ContiguousArrayBuffer<T>?
  func requestNativeBuffer() -> ContiguousArrayBuffer<T>?
  subscript (i: Int) -> T {
    get {}
    set {}
  }
  var count: Int {
    get {}
    set {}
  }
  var capacity: Int {
    get {}
  }
  func _uninitializedCopy(subRange: Range<Int>, target: UnsafePointer<T>) -> UnsafePointer<T>
  subscript (subRange: Range<Int>) -> SliceBuffer<T> {
    get {}
  }
  func isUniquelyReferenced() -> Bool
  func isMutable() -> Bool
  func _asCocoaArray() -> _CocoaArray
  var owner: AnyObject? {
    get {}
  }
  var identity: Word {
    get {}
  }
  func canStoreElementsOfDynamicType(proposedElementType: Any.Type) -> Bool
  func storesOnlyElementsOfType<U>(_: U.Type) -> Bool
  var _storage: ContiguousArrayStorage<T>? {
    get {}
  }
  typealias _Base = HeapBuffer<_ArrayBody, T>
  var _base: HeapBuffer<_ArrayBody, T>
}
extension ContiguousArrayBuffer<T> : Collection {
  var startIndex: Int {
    get {}
  }
  var endIndex: Int {
    get {}
  }
  func generate() -> IndexingGenerator<ContiguousArrayBuffer<T>>
}
```

### ``ContiguousArrayStorage``
```scala
@objc @final class ContiguousArrayStorage<T> : _NSSwiftArray {
  typealias Buffer = ContiguousArrayBuffer<T>
  @objc deinit
  @final @final @final @final func __getInstanceSizeAndAlignMask() -> (Int, Int)
  @final @final @final override func canStoreElementsOfDynamicType(proposedElementType: Any.Type) -> Bool
  @final @final @final override var staticElementType: Any.Type {
    @final @final @final get {}
  }
  init()
}
```

### ``COpaquePointer``
```scala
struct COpaquePointer : Equatable, Hashable, LogicValue {
  var value: RawPointer
  init()
  init(_ v: RawPointer)
  static func null() -> COpaquePointer
  var _isNull: Bool {
    @transparent get {}
  }
  @transparent func getLogicValue() -> Bool
  var hashValue: Int {
    get {}
  }
}
extension COpaquePointer {
  @conversion @transparent func __conversion() -> CMutableVoidPointer
  @conversion @transparent func __conversion() -> CConstVoidPointer
}
extension COpaquePointer {
  init<T>(_ from: UnsafePointer<T>)
}
extension COpaquePointer : CVarArg {
  func encode() -> Word[]
}
```

### ``CShort``
```scala
typealias CShort = Int16
```

### ``CSignedChar``
```scala
typealias CSignedChar = Int8
```

### ``CString``
```scala
struct CString : _BuiltinExtendedGraphemeClusterLiteralConvertible, ExtendedGraphemeClusterLiteralConvertible, _BuiltinStringLiteralConvertible, StringLiteralConvertible, LogicValue {
  var _bytesPtr: UnsafePointer<UInt8>
  @transparent init(_ _bytesPtr: UnsafePointer<UInt8>)
  static func _convertFromBuiltinExtendedGraphemeClusterLiteral(start: RawPointer, byteSize: Word, isASCII: Int1) -> CString
  static func convertFromExtendedGraphemeClusterLiteral(value: CString) -> CString
  static func _convertFromBuiltinStringLiteral(start: RawPointer, byteSize: Word, isASCII: Int1) -> CString
  static func convertFromStringLiteral(value: CString) -> CString
  var _isNull: Bool {
    @transparent get {}
  }
  @transparent func getLogicValue() -> Bool
  func persist() -> CChar[]?
}
extension CString : DebugPrintable {
  var debugDescription: String {
    get {}
  }
}
extension CString : Equatable, Hashable, Comparable {
  var hashValue: Int {
    get {}
  }
}
extension CString : Streamable {
  func writeTo<Target : OutputStream>(inout target: Target)
}
```

### ``CUnsignedChar``
```scala
typealias CUnsignedChar = UInt8
```

### ``CUnsignedInt``
```scala
typealias CUnsignedInt = UInt32
```

### ``CUnsignedLong``
```scala
typealias CUnsignedLong = UInt
```

### ``CUnsignedLongLong``
```scala
typealias CUnsignedLongLong = UInt64
```

### ``CUnsignedShort``
```scala
typealias CUnsignedShort = UInt16
```

### ``CVaListPointer``
```scala
struct CVaListPointer {
  var value: UnsafePointer<Void>
  init(fromUnsafePointer from: UnsafePointer<Void>)
  @conversion func __conversion() -> CMutableVoidPointer
}
```

### ``CVarArg``
```scala
protocol CVarArg {
  func encode() -> Word[]
}
```

### ``CWideChar``
```scala
typealias CWideChar = UnicodeScalar
```

### ``DebugPrintable``
```scala
protocol DebugPrintable {
  var debugDescription: String { get }
}
```

### ``Dictionary``
```scala
struct Dictionary<KeyType : Hashable, ValueType> : Collection, DictionaryLiteralConvertible {
  typealias _Self = Dictionary<KeyType, ValueType>
  typealias _VariantStorage = _VariantDictionaryStorage<KeyType, ValueType>
  typealias _NativeStorage = _NativeDictionaryStorage<KeyType, ValueType>
  typealias Element = (KeyType, ValueType)
  typealias Index = DictionaryIndex<KeyType, ValueType>
  var _variantStorage: _VariantDictionaryStorage<KeyType, ValueType>
  init(minimumCapacity: Int = default)
  init(_nativeStorage: _NativeDictionaryStorage<KeyType, ValueType>)
  init(_cocoaDictionary: _SwiftNSDictionary)
  var startIndex: DictionaryIndex<KeyType, ValueType> {
    get {}
  }
  var endIndex: DictionaryIndex<KeyType, ValueType> {
    get {}
  }
  func indexForKey(key: KeyType) -> DictionaryIndex<KeyType, ValueType>?
  subscript (i: DictionaryIndex<KeyType, ValueType>) -> (KeyType, ValueType) {
    get {}
  }
  subscript (key: KeyType) -> ValueType? {
    get {}
    set(newValue) {}
  }
  func updateValue(value: ValueType, forKey key: KeyType) -> ValueType?
  func removeAtIndex(index: DictionaryIndex<KeyType, ValueType>)
  func removeValueForKey(key: KeyType) -> ValueType?
  func removeAll(keepCapacity: Bool = default)
  var count: Int {
    get {}
  }
  func generate() -> DictionaryGenerator<KeyType, ValueType>
  static func convertFromDictionaryLiteral(elements: (KeyType, ValueType)...) -> Dictionary<KeyType, ValueType>
  var keys: MapCollectionView<Dictionary<KeyType, ValueType>, KeyType> {
    get {}
  }
  var values: MapCollectionView<Dictionary<KeyType, ValueType>, ValueType> {
    get {}
  }
}
extension Dictionary<KeyType, ValueType> : Printable, DebugPrintable {
  func _makeDescription(#isDebug: Bool) -> String
  var description: String {
    get {}
  }
  var debugDescription: String {
    get {}
  }
}
extension Dictionary<KeyType, ValueType> : Reflectable {
  func getMirror() -> Mirror
}
```

### ``DictionaryGenerator``
```scala
enum DictionaryGenerator<KeyType : Hashable, ValueType> : Generator {
  typealias _NativeIndex = _NativeDictionaryIndex<KeyType, ValueType>
  case _Native(start: _NativeDictionaryIndex<KeyType, ValueType>, end: _NativeDictionaryIndex<KeyType, ValueType>)
  case _Cocoa(_CocoaDictionaryGenerator)
  var _guaranteedNative: Bool {
    @transparent get {}
  }
  func _nativeNext() -> (KeyType, ValueType)?
  func next() -> (KeyType, ValueType)?
}
```

### ``DictionaryIndex``
```scala
enum DictionaryIndex<KeyType : Hashable, ValueType> : BidirectionalIndex {
  typealias _NativeIndex = _NativeDictionaryIndex<KeyType, ValueType>
  typealias _CocoaIndex = _CocoaDictionaryIndex
  case _Native(_NativeDictionaryIndex<KeyType, ValueType>)
  case _Cocoa(_CocoaIndex)
  var _guaranteedNative: Bool {
    @transparent get {}
  }
  var _nativeIndex: _NativeDictionaryIndex<KeyType, ValueType> {
    @transparent get {}
  }
  var _cocoaIndex: _CocoaIndex {
    @transparent get {}
  }
  typealias Index = DictionaryIndex<KeyType, ValueType>
  func pred() -> DictionaryIndex<KeyType, ValueType>
  func succ() -> DictionaryIndex<KeyType, ValueType>
}
```

### ``DictionaryLiteralConvertible``
```scala
protocol DictionaryLiteralConvertible {
  typealias Key
  typealias Value
  class func convertFromDictionaryLiteral(elements: (Key, Value)...) -> Self
}
```

### ``Double``
```scala
struct Double {
  var value: FPIEEE64
  @transparent init()
  @transparent init(_ v: FPIEEE64)
  @transparent init(_ value: Double)
}
extension Double : Printable {
  var description: String {
    get {}
  }
}
extension Double : FloatingPointNumber {
  typealias _BitsType = UInt64
  static func _fromBitPattern(bits: _BitsType) -> Double
  func _toBitPattern() -> _BitsType
  func __getSignBit() -> Int
  func __getBiasedExponent() -> _BitsType
  func __getSignificand() -> _BitsType
  static var infinity: Double {
    get {}
  }
  static var NaN: Double {
    get {}
  }
  static var quietNaN: Double {
    get {}
  }
  var isSignMinus: Bool {
    get {}
  }
  var isNormal: Bool {
    get {}
  }
  var isFinite: Bool {
    get {}
  }
  var isZero: Bool {
    get {}
  }
  var isSubnormal: Bool {
    get {}
  }
  var isInfinite: Bool {
    get {}
  }
  var isNaN: Bool {
    get {}
  }
  var isSignaling: Bool {
    get {}
  }
}
extension Double {
  var floatingPointClass: FloatingPointClassification {
    get {}
  }
}
extension Double : _BuiltinIntegerLiteralConvertible, IntegerLiteralConvertible {
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Double
  @transparent static func convertFromIntegerLiteral(value: Int64) -> Double
}
extension Double : _BuiltinFloatLiteralConvertible {
  @transparent static func _convertFromBuiltinFloatLiteral(value: FPIEEE64) -> Double
}
extension Double : FloatLiteralConvertible {
  @transparent static func convertFromFloatLiteral(value: Double) -> Double
}
extension Double : Comparable {
}
extension Double : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Double : AbsoluteValuable {
  @transparent static func abs(x: Double) -> Double
}
extension Double {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
}
extension Double {
  @transparent init(_ v: Float)
  @transparent init(_ v: Float80)
}
extension Double : RandomAccessIndex {
  @transparent func succ() -> Double
  @transparent func pred() -> Double
  @transparent func distanceTo(other: Double) -> Int
  @transparent func advancedBy(amount: Int) -> Double
}
extension Float64 : Reflectable {
  func getMirror() -> Mirror
}
extension Double : CVarArg {
  func encode() -> Word[]
}
```

### ``EmptyCollection``
```scala
struct EmptyCollection<T> : Collection {
  typealias IndexType = Int
  var startIndex: IndexType {
    get {}
  }
  var endIndex: IndexType {
    get {}
  }
  func generate() -> EmptyGenerator<T>
  subscript (i: IndexType) -> T {
    get {}
  }
  init()
}
```

### ``EmptyGenerator``
```scala
struct EmptyGenerator<T> : Generator, Sequence {
  func generate() -> EmptyGenerator<T>
  func next() -> T?
  init()
}
```

### ``EnumerateGenerator``
```scala
struct EnumerateGenerator<Base : Generator> : Generator, Sequence {
  typealias Element = (index: Int, element: Base.Element)
  var base: Base
  var count: Int
  init(_ base: Base)
  func next() -> Element?
  typealias GeneratorType = EnumerateGenerator<Base>
  func generate() -> EnumerateGenerator<Base>
}
```

### ``Equatable``
```scala
protocol Equatable {
  func ==(lhs: Self, rhs: Self) -> Bool
}
```

### ``ExtendedGraphemeClusterLiteralConvertible``
```scala
protocol ExtendedGraphemeClusterLiteralConvertible {
  typealias ExtendedGraphemeClusterLiteralType : _BuiltinExtendedGraphemeClusterLiteralConvertible
  class func convertFromExtendedGraphemeClusterLiteral(value: ExtendedGraphemeClusterLiteralType) -> Self
}
```

### ``ExtendedGraphemeClusterType``
```scala
typealias ExtendedGraphemeClusterType = String
```

### ``ExtensibleCollection``
```scala
protocol ExtensibleCollection : _ExtensibleCollection {
}
```
```scala
protocol _ExtensibleCollection : Collection {
  init()
  func reserveCapacity(n: Self.IndexType.DistanceType)
  func extend<S : Sequence where `Self`.GeneratorType.Element == Self.GeneratorType.Element>(_: S)
}
```

### ``FilterCollectionView``
```scala
struct FilterCollectionView<Base : Collection> : Collection {
  typealias IndexType = FilterCollectionViewIndex<Base>
  var startIndex: FilterCollectionViewIndex<Base> {
    get {}
  }
  var endIndex: FilterCollectionViewIndex<Base> {
    get {}
  }
  subscript (index: FilterCollectionViewIndex<Base>) -> Base.GeneratorType.Element {
    get {}
  }
  func generate() -> FilterGenerator<Base.GeneratorType>
  var _base: Base
  var _include: (Base.GeneratorType.Element) -> Bool
  init(_base: Base, _include: (Base.GeneratorType.Element) -> Bool)
}
```

### ``FilterCollectionViewIndex``
```scala
struct FilterCollectionViewIndex<Base : Collection> : ForwardIndex {
  func succ() -> FilterCollectionViewIndex<Base>
  var _pos: Base.IndexType
  var _end: Base.IndexType
  var _base: Base
  var _include: (Base.GeneratorType.Element) -> Bool
  init(_pos: Base.IndexType, _end: Base.IndexType, _base: Base, _include: (Base.GeneratorType.Element) -> Bool)
}
```

### ``FilterGenerator``
```scala
struct FilterGenerator<Base : Generator> : Generator, Sequence {
  func next() -> Base.Element?
  func generate() -> FilterGenerator<Base>
  var _base: Base
  var _include: (Base.Element) -> Bool
  init(_base: Base, _include: (Base.Element) -> Bool)
}
```

### ``FilterSequenceView``
```scala
struct FilterSequenceView<Base : Sequence> : Sequence {
  func generate() -> FilterGenerator<Base.GeneratorType>
  var _base: Base
  var _include: (Base.GeneratorType.Element) -> Bool
  init(_base: Base, _include: (Base.GeneratorType.Element) -> Bool)
}
```

### ``Float``
```scala
struct Float {
  var value: FPIEEE32
  @transparent init()
  @transparent init(_ v: FPIEEE32)
  @transparent init(_ value: Float)
}
extension Float : Printable {
  var description: String {
    get {}
  }
}
extension Float : FloatingPointNumber {
  typealias _BitsType = UInt32
  static func _fromBitPattern(bits: _BitsType) -> Float
  func _toBitPattern() -> _BitsType
  func __getSignBit() -> Int
  func __getBiasedExponent() -> _BitsType
  func __getSignificand() -> _BitsType
  static var infinity: Float {
    get {}
  }
  static var NaN: Float {
    get {}
  }
  static var quietNaN: Float {
    get {}
  }
  var isSignMinus: Bool {
    get {}
  }
  var isNormal: Bool {
    get {}
  }
  var isFinite: Bool {
    get {}
  }
  var isZero: Bool {
    get {}
  }
  var isSubnormal: Bool {
    get {}
  }
  var isInfinite: Bool {
    get {}
  }
  var isNaN: Bool {
    get {}
  }
  var isSignaling: Bool {
    get {}
  }
}
extension Float {
  var floatingPointClass: FloatingPointClassification {
    get {}
  }
}
extension Float : _BuiltinIntegerLiteralConvertible, IntegerLiteralConvertible {
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Float
  @transparent static func convertFromIntegerLiteral(value: Int64) -> Float
}
extension Float : _BuiltinFloatLiteralConvertible {
  @transparent static func _convertFromBuiltinFloatLiteral(value: FPIEEE64) -> Float
}
extension Float : FloatLiteralConvertible {
  @transparent static func convertFromFloatLiteral(value: Float) -> Float
}
extension Float : Comparable {
}
extension Float : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Float : AbsoluteValuable {
  @transparent static func abs(x: Float) -> Float
}
extension Float {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
}
extension Float {
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension Float : RandomAccessIndex {
  @transparent func succ() -> Float
  @transparent func pred() -> Float
  @transparent func distanceTo(other: Float) -> Int
  @transparent func advancedBy(amount: Int) -> Float
}
extension Float32 : Reflectable {
  func getMirror() -> Mirror
}
extension Float : CVarArg {
  func encode() -> Word[]
}
```

### ``Float32``
```scala
typealias Float32 = Float
```

### ``Float64``
```scala
typealias Float64 = Double
```

### ``Float80``
```scala
struct Float80 {
  var value: FPIEEE80
  @transparent init()
  @transparent init(_ v: FPIEEE80)
  @transparent init(_ value: Float80)
}
extension Float80 : Printable {
  var description: String {
    get {}
  }
}
extension Float80 : _BuiltinIntegerLiteralConvertible, IntegerLiteralConvertible {
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Float80
  @transparent static func convertFromIntegerLiteral(value: Int64) -> Float80
}
extension Float80 : _BuiltinFloatLiteralConvertible {
  @transparent static func _convertFromBuiltinFloatLiteral(value: FPIEEE64) -> Float80
}
extension Float80 : FloatLiteralConvertible {
  @transparent static func convertFromFloatLiteral(value: Float80) -> Float80
}
extension Float80 : Comparable {
}
extension Float80 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Float80 : AbsoluteValuable {
  @transparent static func abs(x: Float80) -> Float80
}
extension Float80 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
}
extension Float80 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
}
extension Float80 : RandomAccessIndex {
  @transparent func succ() -> Float80
  @transparent func pred() -> Float80
  @transparent func distanceTo(other: Float80) -> Int
  @transparent func advancedBy(amount: Int) -> Float80
}
```

### ``FloatingPointClassification``
```scala
enum FloatingPointClassification {
  case SignalingNaN
  case QuietNaN
  case NegativeInfinity
  case NegativeNormal
  case NegativeSubnormal
  case NegativeZero
  case PositiveZero
  case PositiveSubnormal
  case PositiveNormal
  case PositiveInfinity
  var hashValue: Int {
    get {}
  }
}
extension FloatingPointClassification : Equatable {
}
```

### ``FloatingPointNumber``
```scala
protocol FloatingPointNumber {
  typealias _BitsType
  class func _fromBitPattern(bits: _BitsType) -> Self
  func _toBitPattern() -> _BitsType
  class var infinity: Self { get }
  class var NaN: Self { get }
  class var quietNaN: Self { get }
  var floatingPointClass: FloatingPointClassification { get }
  var isSignMinus: Bool { get }
  var isNormal: Bool { get }
  var isFinite: Bool { get }
  var isZero: Bool { get }
  var isSubnormal: Bool { get }
  var isInfinite: Bool { get }
  var isNaN: Bool { get }
  var isSignaling: Bool { get }
}
```

### ``FloatLiteralConvertible``
```scala
protocol FloatLiteralConvertible {
  typealias FloatLiteralType : _BuiltinFloatLiteralConvertible
  class func convertFromFloatLiteral(value: FloatLiteralType) -> Self
}
```

### ``FloatLiteralType``
```scala
typealias FloatLiteralType = Double
```

### ``ForwardIndex``
```scala
protocol ForwardIndex : _ForwardIndex {
  func ~>(start: Self, _: (_Distance, Self)) -> Self.DistanceType
  func ~>(start: Self, _: (_Advance, Self.DistanceType)) -> Self
  func ~>(start: Self, _: (_Advance, (Self.DistanceType, Self))) -> Self
}
```

### ``Generator``
```scala
protocol Generator {
  typealias Element
  func next() -> Element?
}
```

### ``GeneratorOf``
```scala
struct GeneratorOf<T> : Generator, Sequence {
  init(_ next: () -> T?)
  init<G : Generator where T == T>(_ self_: G)
  func next() -> T?
  func generate() -> GeneratorOf<T>
  let _next: () -> T?
}
```

### ``GeneratorOfOne``
```scala
struct GeneratorOfOne<T> : Generator, Sequence {
  init(_ elements: T?)
  func generate() -> GeneratorOfOne<T>
  func next() -> T?
  var elements: T?
}
```

### ``GeneratorSequence``
```scala
struct GeneratorSequence<G : Generator> : Generator, Sequence {
  init(_ base: G)
  func next() -> G.Element?
  func generate() -> GeneratorSequence<G>
  var _base: G
}
```

### ``Hashable``
```scala
protocol Hashable : Equatable {
  var hashValue: Int { get }
}
```

### ``HeapBuffer``
```scala
struct HeapBuffer<Value, Element> : LogicValue, Equatable {
  typealias Storage = HeapBufferStorage<Value, Element>
  let storage: HeapBufferStorage<Value, Element>?
  static func _valueOffset() -> Int
  static func _elementOffset() -> Int
  static func _requiredAlignMask() -> Int
  var _address: UnsafePointer<Int8> {
    get {}
  }
  var _value: UnsafePointer<Value> {
    get {}
  }
  var elementStorage: UnsafePointer<Element> {
    get {}
  }
  func _allocatedSize() -> Int
  func _allocatedAlignMask() -> Int
  func _allocatedSizeAndAlignMask() -> (Int, Int)
  func _capacity() -> Int
  init()
  init(_ storage: HeapBufferStorage<Value, Element>)
  init(_ storageClass: HeapBufferStorageBase.Type, _ initializer: Value, _ capacity: Int)
  var value: Value {
    get {}
    set(newValue) {}
  }
  func getLogicValue() -> Bool
  subscript (i: Int) -> Element {
    get {}
    set(newValue) {}
  }
  @conversion func __conversion() -> NativeObject
  static func fromNativeObject(x: NativeObject) -> HeapBuffer<Value, Element>
  func isUniquelyReferenced() -> Bool
}
```

### ``HeapBufferStorage``
```scala
@objc class HeapBufferStorage<Value, Element> : HeapBufferStorageBase {
  typealias Buffer = HeapBuffer<Value, Element>
  @objc deinit
  func __getInstanceSizeAndAlignMask() -> (Int, Int)
  init()
}
```

### ``HeapBufferStorageBase``
```scala
@objc class HeapBufferStorageBase {
  @objc deinit
  @objc init()
}
```

### ``ImplicitlyUnwrappedOptional``
```scala
struct ImplicitlyUnwrappedOptional<T> : LogicValue, Reflectable {
  var value: T?
  init()
  init(_ v: T?)
  static var None: ImplicitlyUnwrappedOptional<T> {
    @transparent get {}
  }
  @transparent static func Some(value: T) -> ImplicitlyUnwrappedOptional<T>
  @transparent func getLogicValue() -> Bool
  func getMirror() -> Mirror
  func map<U>(f: (T) -> U) -> ImplicitlyUnwrappedOptional<U>
}
extension ImplicitlyUnwrappedOptional<T> : Printable {
  var description: String {
    get {}
  }
}
extension ImplicitlyUnwrappedOptional<T> : _ConditionallyBridgedToObjectiveC {
  typealias ObjectiveCType = AnyObject
  static func getObjectiveCType() -> Any.Type
  func bridgeToObjectiveC() -> AnyObject
  static func bridgeFromObjectiveC(x: AnyObject) -> T!?
  static func isBridgedToObjectiveC() -> Bool
}
```

### ``IndexingGenerator``
```scala
struct IndexingGenerator<C : _Collection> : Generator, Sequence {
  init(_ seq: C)
  func generate() -> IndexingGenerator<C>
  func next() -> C._Element?
  var _elements: C
  var _position: C.IndexType
}
```

### ``IndirectArrayBuffer``
```scala
@final class IndirectArrayBuffer {
  init<T>(nativeBuffer: ContiguousArrayBuffer<T>, isMutable: Bool, needsElementTypeCheck: Bool)
  init(cocoa: _CocoaArray, needsElementTypeCheck: Bool)
  init<Target>(castFrom source: IndirectArrayBuffer, toElementType _: Target.Type)
  @final @final @final func replaceStorage<T>(newBuffer: ContiguousArrayBuffer<T>)
  var buffer: AnyObject?
  var isMutable: Bool
  var isCocoa: Bool
  var needsElementTypeCheck: Bool
  @final @final @final func getNativeBufferOf<T>(_: T.Type) -> ContiguousArrayBuffer<T>
  @final @final @final func getCocoa() -> _CocoaArray
  @objc deinit
}
```

### ``Int``
```scala
struct Int : SignedInteger {
  var value: Word
  @transparent init()
  @transparent init(_ v: Word)
  @transparent init(_ value: Int)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Int
  @transparent static func convertFromIntegerLiteral(value: Int) -> Int
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = Int
  func getArrayBoundValue() -> Int
  static var max: Int {
    @transparent get {}
  }
  static var min: Int {
    @transparent get {}
  }
}
extension Int : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Int : Printable {
  var description: String {
    get {}
  }
}
extension Int : RandomAccessIndex {
  @transparent func succ() -> Int
  @transparent func pred() -> Int
  @transparent func distanceTo(other: Int) -> Int
  @transparent func advancedBy(amount: Int) -> Int
  @transparent static func uncheckedAdd(lhs: Int, _ rhs: Int) -> (Int, Bool)
  @transparent static func uncheckedSubtract(lhs: Int, _ rhs: Int) -> (Int, Bool)
  @transparent static func uncheckedMultiply(lhs: Int, _ rhs: Int) -> (Int, Bool)
  @transparent static func uncheckedDivide(lhs: Int, _ rhs: Int) -> (Int, Bool)
  @transparent static func uncheckedModulus(lhs: Int, _ rhs: Int) -> (Int, Bool)
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: IntMax) -> Int
}
extension Int : SignedNumber {
}
extension Int {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent func asUnsigned() -> UInt
}
extension Int : BitwiseOperations {
  static var allZeros: Int {
    @transparent get {}
  }
}
extension Int {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension Int : Reflectable {
  func getMirror() -> Mirror
}
extension Int : CVarArg {
  func encode() -> Word[]
}
```

### ``Int16``
```scala
struct Int16 : SignedInteger {
  var value: Int16
  @transparent init()
  @transparent init(_ v: Int16)
  @transparent init(_ value: Int16)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Int16
  @transparent static func convertFromIntegerLiteral(value: Int16) -> Int16
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = Int16
  func getArrayBoundValue() -> Int16
  static var max: Int16 {
    @transparent get {}
  }
  static var min: Int16 {
    @transparent get {}
  }
}
extension Int16 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Int16 : Printable {
  var description: String {
    get {}
  }
}
extension Int16 : RandomAccessIndex {
  @transparent func succ() -> Int16
  @transparent func pred() -> Int16
  @transparent func distanceTo(other: Int16) -> Int
  @transparent func advancedBy(amount: Int) -> Int16
  @transparent static func uncheckedAdd(lhs: Int16, _ rhs: Int16) -> (Int16, Bool)
  @transparent static func uncheckedSubtract(lhs: Int16, _ rhs: Int16) -> (Int16, Bool)
  @transparent static func uncheckedMultiply(lhs: Int16, _ rhs: Int16) -> (Int16, Bool)
  @transparent static func uncheckedDivide(lhs: Int16, _ rhs: Int16) -> (Int16, Bool)
  @transparent static func uncheckedModulus(lhs: Int16, _ rhs: Int16) -> (Int16, Bool)
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: IntMax) -> Int16
}
extension Int16 : SignedNumber {
}
extension Int16 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asUnsigned() -> UInt16
}
extension Int16 : BitwiseOperations {
  static var allZeros: Int16 {
    @transparent get {}
  }
}
extension Int16 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension Int16 : Reflectable {
  func getMirror() -> Mirror
}
extension Int16 : CVarArg {
  func encode() -> Word[]
}
```

### ``Int32``
```scala
struct Int32 : SignedInteger {
  var value: Int32
  @transparent init()
  @transparent init(_ v: Int32)
  @transparent init(_ value: Int32)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Int32
  @transparent static func convertFromIntegerLiteral(value: Int32) -> Int32
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = Int32
  func getArrayBoundValue() -> Int32
  static var max: Int32 {
    @transparent get {}
  }
  static var min: Int32 {
    @transparent get {}
  }
}
extension Int32 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Int32 : Printable {
  var description: String {
    get {}
  }
}
extension Int32 : RandomAccessIndex {
  @transparent func succ() -> Int32
  @transparent func pred() -> Int32
  @transparent func distanceTo(other: Int32) -> Int
  @transparent func advancedBy(amount: Int) -> Int32
  @transparent static func uncheckedAdd(lhs: Int32, _ rhs: Int32) -> (Int32, Bool)
  @transparent static func uncheckedSubtract(lhs: Int32, _ rhs: Int32) -> (Int32, Bool)
  @transparent static func uncheckedMultiply(lhs: Int32, _ rhs: Int32) -> (Int32, Bool)
  @transparent static func uncheckedDivide(lhs: Int32, _ rhs: Int32) -> (Int32, Bool)
  @transparent static func uncheckedModulus(lhs: Int32, _ rhs: Int32) -> (Int32, Bool)
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: IntMax) -> Int32
}
extension Int32 : SignedNumber {
}
extension Int32 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asUnsigned() -> UInt32
}
extension Int32 : BitwiseOperations {
  static var allZeros: Int32 {
    @transparent get {}
  }
}
extension Int32 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension Int32 : Reflectable {
  func getMirror() -> Mirror
}
extension Int32 : CVarArg {
  func encode() -> Word[]
}
```

### ``Int64``
```scala
struct Int64 : SignedInteger {
  var value: Int64
  @transparent init()
  @transparent init(_ v: Int64)
  @transparent init(_ value: Int64)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Int64
  @transparent static func convertFromIntegerLiteral(value: Int64) -> Int64
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = Int64
  func getArrayBoundValue() -> Int64
  static var max: Int64 {
    @transparent get {}
  }
  static var min: Int64 {
    @transparent get {}
  }
}
extension Int64 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Int64 : Printable {
  var description: String {
    get {}
  }
}
extension Int64 : RandomAccessIndex {
  @transparent func succ() -> Int64
  @transparent func pred() -> Int64
  @transparent func distanceTo(other: Int64) -> Int
  @transparent func advancedBy(amount: Int) -> Int64
  @transparent static func uncheckedAdd(lhs: Int64, _ rhs: Int64) -> (Int64, Bool)
  @transparent static func uncheckedSubtract(lhs: Int64, _ rhs: Int64) -> (Int64, Bool)
  @transparent static func uncheckedMultiply(lhs: Int64, _ rhs: Int64) -> (Int64, Bool)
  @transparent static func uncheckedDivide(lhs: Int64, _ rhs: Int64) -> (Int64, Bool)
  @transparent static func uncheckedModulus(lhs: Int64, _ rhs: Int64) -> (Int64, Bool)
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: IntMax) -> Int64
}
extension Int64 : SignedNumber {
}
extension Int64 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asUnsigned() -> UInt64
}
extension Int64 : BitwiseOperations {
  static var allZeros: Int64 {
    @transparent get {}
  }
}
extension Int64 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension Int64 : Reflectable {
  func getMirror() -> Mirror
}
extension Int64 : CVarArg {
  func encode() -> Word[]
}
```

### ``Int8``
```scala
struct Int8 : SignedInteger {
  var value: Int8
  @transparent init()
  @transparent init(_ v: Int8)
  @transparent init(_ value: Int8)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> Int8
  @transparent static func convertFromIntegerLiteral(value: Int8) -> Int8
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = Int8
  func getArrayBoundValue() -> Int8
  static var max: Int8 {
    @transparent get {}
  }
  static var min: Int8 {
    @transparent get {}
  }
}
extension Int8 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension Int8 : Printable {
  var description: String {
    get {}
  }
}
extension Int8 : RandomAccessIndex {
  @transparent func succ() -> Int8
  @transparent func pred() -> Int8
  @transparent func distanceTo(other: Int8) -> Int
  @transparent func advancedBy(amount: Int) -> Int8
  @transparent static func uncheckedAdd(lhs: Int8, _ rhs: Int8) -> (Int8, Bool)
  @transparent static func uncheckedSubtract(lhs: Int8, _ rhs: Int8) -> (Int8, Bool)
  @transparent static func uncheckedMultiply(lhs: Int8, _ rhs: Int8) -> (Int8, Bool)
  @transparent static func uncheckedDivide(lhs: Int8, _ rhs: Int8) -> (Int8, Bool)
  @transparent static func uncheckedModulus(lhs: Int8, _ rhs: Int8) -> (Int8, Bool)
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: IntMax) -> Int8
}
extension Int8 : SignedNumber {
}
extension Int8 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asUnsigned() -> UInt8
}
extension Int8 : BitwiseOperations {
  static var allZeros: Int8 {
    @transparent get {}
  }
}
extension Int8 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension Int8 : Reflectable {
  func getMirror() -> Mirror
}
extension Int8 : CVarArg {
  func encode() -> Word[]
}
```

### ``Integer``
```scala
protocol Integer : _Integer, RandomAccessIndex {
}

protocol _Integer : _BuiltinIntegerLiteralConvertible, IntegerLiteralConvertible, Printable, ArrayBound, Hashable, IntegerArithmetic, BitwiseOperations, _Incrementable {
}
```

### ``IntegerArithmetic``
```scala
protocol IntegerArithmetic : _IntegerArithmetic, Comparable {
  func +(lhs: Self, rhs: Self) -> Self
  func -(lhs: Self, rhs: Self) -> Self
  func *(lhs: Self, rhs: Self) -> Self
  func /(lhs: Self, rhs: Self) -> Self
  func %(lhs: Self, rhs: Self) -> Self
  func toIntMax() -> IntMax
}
```

### ``IntegerLiteralConvertible``
```scala
protocol IntegerLiteralConvertible {
  typealias IntegerLiteralType : _BuiltinIntegerLiteralConvertible
  class func convertFromIntegerLiteral(value: IntegerLiteralType) -> Self
}
```

### ``IntegerLiteralType``
```scala
typealias IntegerLiteralType = Int
```

### ``IntEncoder``
```scala
struct IntEncoder : Sink {
  var asInt: UInt64
  var shift: UInt64
  func put(x: CodeUnit)
  init(asInt: UInt64, shift: UInt64)
  init()
}
```

### ``IntMax``
```scala
typealias IntMax = Int64
```

### ``Less``
```scala
struct Less<T : Comparable> {
  static func compare(x: T, _ y: T) -> Bool
  init()
}
```

### ``LifetimeManager``
```scala
class LifetimeManager {
  var _managedRefs: NativeObject[]
  var _releaseCalled: Bool
  init()
  @objc deinit
  func put(objPtr: NativeObject)
  func release()
}
```

### ``LogicValue``
```scala
protocol LogicValue {
  func getLogicValue() -> Bool
}
```

### ``MapCollectionView``
```scala
struct MapCollectionView<Base : Collection, T> : Collection {
  var startIndex: Base.IndexType {
    get {}
  }
  var endIndex: Base.IndexType {
    get {}
  }
  subscript (index: Base.IndexType) -> T {
    get {}
  }
  func generate() -> MapSequenceGenerator<Base.GeneratorType, T>
  var _base: Base
  var _transform: (Base.GeneratorType.Element) -> T
  init(_base: Base, _transform: (Base.GeneratorType.Element) -> T)
}
```

### ``MapSequenceGenerator``
```scala
struct MapSequenceGenerator<Base : Generator, T> : Generator, Sequence {
  func next() -> T?
  func generate() -> MapSequenceGenerator<Base, T>
  var _base: Base
  var _transform: (Base.Element) -> T
  init(_base: Base, _transform: (Base.Element) -> T)
}
```

### ``MapSequenceView``
```scala
struct MapSequenceView<Base : Sequence, T> : Sequence {
  func generate() -> MapSequenceGenerator<Base.GeneratorType, T>
  var _base: Base
  var _transform: (Base.GeneratorType.Element) -> T
  init(_base: Base, _transform: (Base.GeneratorType.Element) -> T)
}
```

### ``MaxBuiltinFloatType``
```scala
typealias MaxBuiltinFloatType = FPIEEE64
```

### ``MaxBuiltinIntegerType``
```scala
typealias MaxBuiltinIntegerType = Int2048
```

### ``Mirror``
```scala
protocol Mirror {
  var value: Any { get }
  var valueType: Any.Type { get }
  var objectIdentifier: ObjectIdentifier? { get }
  var count: Int { get }
  subscript (i: Int) -> (String, Mirror) { get }
  var summary: String { get }
  var quickLookObject: QuickLookObject? { get }
  var disposition: MirrorDisposition { get }
}
```

### ``MirrorDisposition``
```scala
enum MirrorDisposition {
  case Struct
  case Class
  case Enum
  case Tuple
  case Aggregate
  case IndexContainer
  case KeyContainer
  case MembershipContainer
  case Container
  case Optional
  var hashValue: Int {
    get {}
  }
}
```

### ``MutableCollection``
```scala
protocol MutableCollection : Collection {
  subscript (i: Self.IndexType) -> Self.GeneratorType.Element { get set }
}
```

### ``MutableSliceable``
```scala
protocol MutableSliceable : Sliceable, MutableCollection {
  subscript (_: Range<Self.IndexType>) -> Self.SliceType { get set }
}
```

### ``ObjectIdentifier``
```scala
struct ObjectIdentifier : Hashable {
  let value: RawPointer
  func uintValue() -> UInt
  var hashValue: Int {
    get {}
  }
  init(_ x: AnyObject)
}
```

### ``OnHeap``
```scala
struct OnHeap<T> {
  typealias Buffer = HeapBuffer<T, Void>
  init(_ value: T)
  @conversion func __conversion() -> T
  var _storage: HeapBuffer<T, Void>
}
```

### ``Optional``
This is the alias of Type? .
```scala
enum Optional<T> : LogicValue, Reflectable {
  case None
  case Some(T)
  init()
  init(_ some: T)
  @transparent func getLogicValue() -> Bool
  func map<U>(f: (T) -> U) -> U?
  func getMirror() -> Mirror
}
extension Optional<T> : Printable {
  var description: String {
    get {}
  }
}
```

### ``OutputStream``
```scala
protocol OutputStream {
  func write(string: String)
}
```

### ``PermutationGenerator``
```scala
struct PermutationGenerator<C : Collection, Indices : Sequence where C.IndexType == C.IndexType> : Generator, Sequence {
  var seq: C
  var indices: Indices.GeneratorType
  typealias Element = C.GeneratorType.Element
  func next() -> Element?
  typealias GeneratorType = PermutationGenerator<C, Indices>
  func generate() -> PermutationGenerator<C, Indices>
  init(elements seq: C, indices: Indices)
}
```

### ``Printable``
```scala
protocol Printable {
  var description: String { get }
}
```

### ``QuickLookObject``
```scala
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

### ``RandomAccessIndex``
```scala
protocol RandomAccessIndex : BidirectionalIndex, _RandomAccessIndex {}
protocol _RandomAccessIndex : _BidirectionalIndex {
  func distanceTo(_: Self) -> Self.DistanceType
  func advancedBy(_: Self.DistanceType) -> Self
}
```

### ``Range``
```scala
struct Range<T : ForwardIndex> : LogicValue, Sliceable {
  @transparent init(start: T, end: T)
  var isEmpty: Bool {
    get {}
  }
  func getLogicValue() -> Bool
  subscript (i: T) -> T {
    get {}
  }
  subscript (x: Range<T>) -> Range<T> {
    get {}
  }
  typealias GeneratorType = RangeGenerator<T>
  func generate() -> RangeGenerator<T>
  func by(stride: T.DistanceType) -> StridedRangeGenerator<T>
  var startIndex: T {
    get {}
    set(newValue) {}
  }
  var endIndex: T {
    get {}
    set(newValue) {}
  }
  var _startIndex: T
  var _endIndex: T
}
```

### ``RangeGenerator``
```scala
struct RangeGenerator<T : ForwardIndex> : Generator, Sequence {
  typealias Element = T
  @transparent init(_ bounds: Range<T>)
  func next() -> T?
  typealias GeneratorType = RangeGenerator<T>
  func generate() -> RangeGenerator<T>
  var startIndex: T
  var endIndex: T
}
```

### ``RawByte``
```scala
struct RawByte {
  let _inaccessible: UInt8
  init(_inaccessible: UInt8)
}
```

### ``RawOptionSet``
```scala
protocol RawOptionSet : _RawOptionSet, LogicValue, Equatable {
  class func fromMask(raw: Self.RawType) -> Self
}
```

### ``RawRepresentable``
```scala
protocol RawRepresentable {
  typealias RawType
  class func fromRaw(raw: RawType) -> Self?
  func toRaw() -> RawType
}
```

### ``Reflectable``
```scala
protocol Reflectable {
  func getMirror() -> Mirror
}
```

### ``Repeat``
```scala
struct Repeat<T> : Collection {
  typealias IndexType = Int
  init(count: Int, repeatedValue: T)
  var startIndex: IndexType {
    get {}
  }
  var endIndex: IndexType {
    get {}
  }
  func generate() -> IndexingGenerator<Repeat<T>>
  subscript (i: Int) -> T {
    get {}
  }
  var count: Int
  let repeatedValue: T
}
```

### ``ReverseIndex``
```scala
struct ReverseIndex<I : BidirectionalIndex> : BidirectionalIndex {
  var _base: I
  init(_ _base: I)
  func succ() -> ReverseIndex<I>
  func pred() -> ReverseIndex<I>
}
```

### ``ReverseRange``
```scala
struct ReverseRange<T : BidirectionalIndex> : Sequence {
  init(start: T, pastEnd: T)
  init(range fwd: Range<T>)
  func isEmpty() -> Bool
  func bounds() -> (T, T)
  typealias GeneratorType = ReverseRangeGenerator<T>
  func generate() -> ReverseRangeGenerator<T>
  var _bounds: (T, T)
}
```

### ``ReverseRangeGenerator``
```scala
struct ReverseRangeGenerator<T : BidirectionalIndex> : Generator, Sequence {
  typealias Element = T
  @transparent init(start: T, pastEnd: T)
  func next() -> T?
  typealias GeneratorType = ReverseRangeGenerator<T>
  func generate() -> ReverseRangeGenerator<T>
  var _bounds: (T, T)
}
```

### ``ReverseView``
```scala
struct ReverseView<T : Collection where T.IndexType : BidirectionalIndex> : Collection {
  typealias IndexType = ReverseIndex<T.IndexType>
  typealias GeneratorType = IndexingGenerator<ReverseView<T>>
  init(_ _base: T)
  func generate() -> IndexingGenerator<ReverseView<T>>
  var startIndex: IndexType {
    get {}
  }
  var endIndex: IndexType {
    get {}
  }
  subscript (i: IndexType) -> T.GeneratorType.Element {
    get {}
  }
  var _base: T
}
```

### ``Sequence``
```scala
protocol Sequence : _Sequence_ {
  typealias GeneratorType : Generator
  func generate() -> GeneratorType
  func ~>(_: Self, _: (_UnderestimateCount, ())) -> Int
  func ~><R>(_: Self, _: (_PreprocessingPass, ((Self) -> R))) -> R?
  func ~>(_: Self, _: (_CopyToNativeArrayBuffer, ())) -> ContiguousArrayBuffer<Self.GeneratorType.Element>
}
```

### ``SequenceOf``
```scala
struct SequenceOf<T> : Sequence {
  init<G : Generator where T == T>(_ generate: () -> G)
  init<S : Sequence where T == T>(_ self_: S)
  func generate() -> GeneratorOf<T>
  let _generate: () -> GeneratorOf<T>
}
```

### ``SignedInteger``
```scala
protocol SignedInteger : _SignedInteger, Integer {
}
protocol _SignedInteger : _Integer, SignedNumber {
  func toIntMax() -> IntMax
  class func from(_: IntMax) -> Self
}
```

### ``SignedNumber``
```scala
protocol SignedNumber : _SignedNumber {
  func -(x: Self) -> Self
  func ~>(_: Self, _: (_Abs, ())) -> Self
}
```

### ``Sink``
```scala
protocol Sink {
  typealias Element
  func put(x: Element)
}
```

### ``SinkOf``
```scala
struct SinkOf<T> : Sink {
  init(_ put: (T) -> ())
  init<S : Sink where T == T>(_ base: S)
  func put(x: T)
  let _put: (T) -> ()
}
```

### ``Slice``
```scala
struct Slice<T> : MutableCollection, Sliceable {
  typealias Element = T
  var startIndex: Int {
    get {}
  }
  var endIndex: Int {
    get {}
  }
  subscript (index: Int) -> T {
    get {}
    set {}
  }
  func generate() -> IndexingGenerator<Slice<T>>
  typealias SliceType = Slice<T>
  subscript (subRange: Range<Int>) -> Slice<T> {
    get {}
    set(rhs) {}
  }
  typealias _Buffer = SliceBuffer<T>
  init(_ buffer: SliceBuffer<T>)
  var _buffer: SliceBuffer<T>
}
extension Slice<T> : ArrayLiteralConvertible {
  static func convertFromArrayLiteral(elements: T...) -> Slice<T>
}
extension Slice<T> {
  func _asCocoaArray() -> _CocoaArray
}
extension Slice<T> : ArrayType {
  init()
  init<S : Sequence where T == T>(_ s: S)
  init(count: Int, repeatedValue: T)
  var count: Int {
    get {}
  }
  var capacity: Int {
    get {}
  }
  var isEmpty: Bool {
    get {}
  }
  var _owner: AnyObject? {
    get {}
  }
  var _elementStorageIfContiguous: UnsafePointer<T> {
    get {}
  }
  var _elementStorage: UnsafePointer<T> {
    get {}
  }
  func copy() -> Slice<T>
  func unshare()
  func reserveCapacity(minimumCapacity: Int)
  func append(newElement: T)
  func extend<S : Sequence where T == T>(sequence: S)
  func removeLast() -> T
  func insert(newElement: T, atIndex: Int)
  func removeAtIndex(index: Int) -> T
  func removeAll(keepCapacity: Bool = default)
  func join<S : Sequence where Slice<T> == Slice<T>>(elements: S) -> Slice<T>
  func reduce<U>(initial: U, combine: (U, T) -> U) -> U
  func sort(isOrderedBefore: (T, T) -> Bool)
  func map<U>(transform: (T) -> U) -> Slice<U>
  func reverse() -> Slice<T>
  func filter(includeElement: (T) -> Bool) -> Slice<T>
}
extension Slice<T> : Reflectable {
  func getMirror() -> Mirror
}
extension Slice<T> : Printable, DebugPrintable {
  func _makeDescription(#isDebug: Bool) -> String
  var description: String {
    get {}
  }
  var debugDescription: String {
    get {}
  }
}
extension Slice<T> {
  @transparent func _cPointerArgs() -> (AnyObject?, RawPointer)
  @conversion @transparent func __conversion() -> CConstPointer<T>
  @conversion @transparent func __conversion() -> CConstVoidPointer
}
extension Slice<T> {
  func withUnsafePointerToElements<R>(body: (UnsafePointer<T>) -> R) -> R
}
extension Slice<T> {
  func replaceRange<C : Collection where T == T>(subRange: Range<Int>, with newValues: C)
}
```

### ``Sliceable``
```scala
protocol Sliceable : _Sliceable {
  typealias SliceType : _Sliceable
  subscript (_: Range<Self.IndexType>) -> SliceType { get }
}
```

### ``SliceBuffer``
```scala
struct SliceBuffer<T> : ArrayBufferType {
  typealias Element = T
  typealias NativeStorage = ContiguousArrayStorage<T>
  typealias NativeBuffer = ContiguousArrayBuffer<T>
  init(owner: AnyObject?, start: UnsafePointer<T>, count: Int, hasNativeBuffer: Bool)
  init()
  init(_ buffer: ContiguousArrayBuffer<T>)
  func _invariantCheck()
  var _hasNativeBuffer: Bool {
    get {}
  }
  var nativeBuffer: ContiguousArrayBuffer<T> {
    get {}
  }
  var identity: Word {
    get {}
  }
  var owner: AnyObject?
  var start: UnsafePointer<T>
  var _countAndFlags: UInt
  func _asCocoaArray() -> _CocoaArray
  func requestUniqueMutableBuffer(minimumCapacity: Int) -> ContiguousArrayBuffer<T>?
  func requestNativeBuffer() -> ContiguousArrayBuffer<T>?
  func _uninitializedCopy(subRange: Range<Int>, target: UnsafePointer<T>) -> UnsafePointer<T>
  var elementStorage: UnsafePointer<T> {
    get {}
  }
  var count: Int {
    get {}
    set {}
  }
  var capacity: Int {
    get {}
  }
  func isUniquelyReferenced() -> Bool
  subscript (i: Int) -> T {
    get {}
    set {}
  }
  subscript (subRange: Range<Int>) -> SliceBuffer<T> {
    get {}
  }
  var startIndex: Int {
    get {}
  }
  var endIndex: Int {
    get {}
  }
  func generate() -> IndexingGenerator<SliceBuffer<T>>
  func withUnsafePointerToElements<R>(body: (UnsafePointer<T>) -> R) -> R
}
```

### ``StaticString``
```scala
struct StaticString : _BuiltinExtendedGraphemeClusterLiteralConvertible, ExtendedGraphemeClusterLiteralConvertible, _BuiltinStringLiteralConvertible, StringLiteralConvertible {
  var start: RawPointer
  var byteSize: Word
  var isASCII: Int1
  init()
  init(start: RawPointer, byteSize: Word, isASCII: Int1)
  static func _convertFromBuiltinExtendedGraphemeClusterLiteral(start: RawPointer, byteSize: Word, isASCII: Int1) -> StaticString
  static func convertFromExtendedGraphemeClusterLiteral(value: StaticString) -> StaticString
  static func _convertFromBuiltinStringLiteral(start: RawPointer, byteSize: Word, isASCII: Int1) -> StaticString
  static func convertFromStringLiteral(value: StaticString) -> StaticString
}
```

### ``Streamable``
```scala
protocol Streamable {
  func writeTo<Target : OutputStream>(inout target: Target)
}
```

### ``StridedRangeGenerator``
```scala
struct StridedRangeGenerator<T : ForwardIndex> : Generator, Sequence {
  typealias Element = T
  @transparent init(_ bounds: Range<T>, stride: T.DistanceType)
  func next() -> T?
  typealias GeneratorType = StridedRangeGenerator<T>
  func generate() -> StridedRangeGenerator<T>
  var _bounds: Range<T>
  var _stride: T.DistanceType
}
```

### ``String``
```scala
struct String {
  init()
  init(_ _core: _StringCore)
  var core: _StringCore
}
extension String {
  static func fromCString(cs: CString) -> String
  static func fromCString(up: UnsafePointer<CChar>) -> String
}
extension String {
  init(_ c: Character)
}
extension String : ExtensibleCollection {
  func reserveCapacity(n: Int)
  func extend<S : Sequence where Character == Character>(seq: S)
}
extension String {
  func withCString<Result>(f: (CString) -> Result) -> Result
  func withCString<Result>(f: (UnsafePointer<CChar>) -> Result) -> Result
}
extension String : Reflectable {
  func getMirror() -> Mirror
}
extension String : OutputStream {
  func write(string: String)
}
extension String : Streamable {
  func writeTo<Target : OutputStream>(inout target: Target)
}
extension String : _BuiltinExtendedGraphemeClusterLiteralConvertible {
  static func _convertFromBuiltinExtendedGraphemeClusterLiteral(start: RawPointer, byteSize: Word, isASCII: Int1) -> String
}
extension String : ExtendedGraphemeClusterLiteralConvertible {
  static func convertFromExtendedGraphemeClusterLiteral(value: String) -> String
}
extension String : _BuiltinUTF16StringLiteralConvertible {
  static func _convertFromBuiltinUTF16StringLiteral(start: RawPointer, numberOfCodeUnits: Word) -> String
}
extension String : _BuiltinStringLiteralConvertible {
  static func _convertFromBuiltinStringLiteral(start: RawPointer, byteSize: Word, isASCII: Int1) -> String
}
extension String : StringLiteralConvertible {
  static func convertFromStringLiteral(value: String) -> String
}
extension String : DebugPrintable {
  var debugDescription: String {
    get {}
  }
}
extension String {
  func _encodedLength<Encoding : UnicodeCodec>(encoding: Encoding.Type) -> Int
  func _encode<Encoding : UnicodeCodec, Output : Sink where Encoding.CodeUnit == Encoding.CodeUnit>(encoding: Encoding.Type, output: Output)
}
extension String : Equatable {
}
extension String {
  func _append(rhs: String)
  func _append(x: UnicodeScalar)
  var _utf16Count: Int {
    get {}
  }
  init(_ storage: _StringBuffer)
}
extension String : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension String : StringInterpolationConvertible {
  static func convertFromStringInterpolation(strings: String...) -> String
  static func convertFromStringInterpolationSegment<T>(expr: T) -> String
}
extension String : Comparable {
}
extension String {
  @asmname("swift_stringFromUTF8InRawMemory") static func _fromUTF8InRawMemory(resultStorage: UnsafePointer<String>, start: UnsafePointer<CodeUnit>, utf8Count: Int)
}
extension String : Collection {
  struct Index : BidirectionalIndex {
    init(_ _base: String.UnicodeScalarView.IndexType)
    func succ() -> String.Index
    func pred() -> String.Index
    let _base: String.UnicodeScalarView.IndexType
    var _utf16Index: Int {
      get {}
    }
  }
  var startIndex: String.Index {
    get {}
  }
  var endIndex: String.Index {
    get {}
  }
  subscript (i: String.Index) -> Character {
    get {}
  }
  func generate() -> IndexingGenerator<String>
}
extension String : Sliceable {
  subscript (subRange: Range<String.Index>) -> String {
    get {}
  }
}
extension String {
  func join<S : Sequence where String == String>(elements: S) -> String
}
extension String {
  init<Encoding : UnicodeCodec, Input : Collection where Encoding.CodeUnit == Encoding.CodeUnit>(_ _encoding: Encoding.Type, input: Input)
  init(count sz: Int, repeatedValue c: Character)
  init(count: Int, repeatedValue c: UnicodeScalar)
  var _lines: String[] {
    get {}
  }
  func _split(separator: UnicodeScalar) -> String[]
  var isEmpty: Bool {
    get {}
  }
}
extension String {
  var uppercaseString: String {
    get {}
  }
  var lowercaseString: String {
    get {}
  }
  init(_ _c: UnicodeScalar)
  func _isAll(predicate: (UnicodeScalar) -> Bool) -> Bool
  func hasPrefix(prefix: String) -> Bool
  func hasSuffix(suffix: String) -> Bool
  func _isAlpha() -> Bool
  func _isDigit() -> Bool
  func _isSpace() -> Bool
}
extension String {
  init(_ v: Int64, radix: Int = default, _uppercase: Bool = default)
  init(_ v: UInt64, radix: Int = default, _uppercase: Bool = default)
  init(_ v: Int8, radix: Int = default, _uppercase: Bool = default)
  init(_ v: Int16, radix: Int = default, _uppercase: Bool = default)
  init(_ v: Int32, radix: Int = default, _uppercase: Bool = default)
  init(_ v: Int, radix: Int = default, _uppercase: Bool = default)
  init(_ v: UInt8, radix: Int = default, _uppercase: Bool = default)
  init(_ v: UInt16, radix: Int = default, _uppercase: Bool = default)
  init(_ v: UInt32, radix: Int = default, _uppercase: Bool = default)
  init(_ v: UInt, radix: Int = default, _uppercase: Bool = default)
  typealias _Double = Double
  typealias _Float = Float
  typealias _Bool = Bool
  init(_ v: _Double)
  init(_ v: _Float)
  init(_ b: _Bool)
}
extension String {
  func toInt() -> Int?
}
extension String {
  func _substr(start: Int) -> String
  func _splitFirst(delim: UnicodeScalar) -> (before: String, after: String, wasFound: Bool)
  func _splitFirstIf(pred: (UnicodeScalar) -> Bool) -> (before: String, found: UnicodeScalar, after: String, wasFound: Bool)
  func _splitIf(pred: (UnicodeScalar) -> Bool) -> String[]
}
extension String {
  struct UTF16View : Sliceable {
    var startIndex: Int {
      get {}
    }
    var endIndex: Int {
      get {}
    }
    typealias _GeneratorType = IndexingGenerator<_StringCore>
    typealias GeneratorType = _GeneratorType
    func generate() -> GeneratorType
    subscript (i: Int) -> CodeUnit {
      get {}
    }
    subscript (subRange: Range<Int>) -> String.UTF16View {
      get {}
    }
    init(_ _core: _StringCore)
    let _core: _StringCore
  }
  var utf16: String.UTF16View {
    get {}
  }
}
extension String {
  struct UTF8View : Collection {
    let _core: _StringCore
    init(_ _core: _StringCore)
    struct Index : ForwardIndex {
      init(_ _core: _StringCore, _ _coreIndex: Int, _ _buffer: UTF8Chunk)
      func succ() -> String.UTF8View.Index
      let _core: _StringCore
      let _coreIndex: Int
      let _buffer: UTF8Chunk
    }
    var startIndex: String.UTF8View.Index {
      get {}
    }
    var endIndex: String.UTF8View.Index {
      get {}
    }
    subscript (i: String.UTF8View.Index) -> CodeUnit {
      get {}
    }
    func generate() -> IndexingGenerator<String.UTF8View>
  }
  var utf8: String.UTF8View {
    get {}
  }
  var _contiguousUTF8: UnsafePointer<CodeUnit> {
    get {}
  }
  var nulTerminatedUTF8: ContiguousArray<CodeUnit> {
    get {}
  }
}
extension String {
  struct UnicodeScalarView : Sliceable, Sequence {
    init(_ _base: _StringCore)
    struct ScratchGenerator : Generator {
      var base: _StringCore
      var idx: Int
      init(_ core: _StringCore, _ pos: Int)
      func next() -> CodeUnit?
    }
    struct IndexType : BidirectionalIndex {
      init(_ _position: Int, _ _base: _StringCore)
      func succ() -> String.UnicodeScalarView.IndexType
      func pred() -> String.UnicodeScalarView.IndexType
      var _position: Int
      var _base: _StringCore
    }
    var startIndex: String.UnicodeScalarView.IndexType {
      get {}
    }
    var endIndex: String.UnicodeScalarView.IndexType {
      get {}
    }
    subscript (i: String.UnicodeScalarView.IndexType) -> UnicodeScalar {
      get {}
    }
    func __slice__(start: String.UnicodeScalarView.IndexType, end: String.UnicodeScalarView.IndexType) -> String.UnicodeScalarView
    subscript (r: Range<String.UnicodeScalarView.IndexType>) -> String.UnicodeScalarView {
      get {}
    }
    struct GeneratorType : Generator {
      init(_ _base: IndexingGenerator<_StringCore>)
      func next() -> UnicodeScalar?
      var _base: IndexingGenerator<_StringCore>
    }
    func generate() -> String.UnicodeScalarView.GeneratorType
    @conversion func __conversion() -> String
    func compare(other: String.UnicodeScalarView) -> Int
    func _compareUnicode(other: String.UnicodeScalarView) -> Int
    var _base: _StringCore
  }
}
extension String {
  func compare(other: String) -> Int
  var unicodeScalars: String.UnicodeScalarView {
    get {}
  }
}
```

### ``StringElement``
```scala
protocol StringElement {
  class func toUTF16CodeUnit(_: Self) -> CodeUnit
  class func fromUTF16CodeUnit(utf16: CodeUnit) -> Self
}
```

### ``StringInterpolationConvertible``
```scala
protocol StringInterpolationConvertible {
  class func convertFromStringInterpolation(strings: Self...) -> Self
  class func convertFromStringInterpolationSegment<T>(expr: T) -> Self
}
```

### ``StringLiteralConvertible``
```scala
protocol StringLiteralConvertible : ExtendedGraphemeClusterLiteralConvertible {
  typealias StringLiteralType : _BuiltinStringLiteralConvertible
  class func convertFromStringLiteral(value: StringLiteralType) -> Self
}
```

### ``StringLiteralType``
```scala
typealias StringLiteralType = String
```

### ``UInt``
```scala
struct UInt : UnsignedInteger {
  var value: Word
  @transparent init()
  @transparent init(_ v: Word)
  @transparent init(_ value: UInt)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> UInt
  @transparent static func convertFromIntegerLiteral(value: UInt) -> UInt
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = UInt
  func getArrayBoundValue() -> UInt
  static var max: UInt {
    @transparent get {}
  }
  static var min: UInt {
    @transparent get {}
  }
}
extension UInt : Printable {
  var description: String {
    get {}
  }
}
extension UInt : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension UInt : RandomAccessIndex {
  @transparent func succ() -> UInt
  @transparent func pred() -> UInt
  @transparent func distanceTo(other: UInt) -> Int
  @transparent func advancedBy(amount: Int) -> UInt
  @transparent static func uncheckedAdd(lhs: UInt, _ rhs: UInt) -> (UInt, Bool)
  @transparent static func uncheckedSubtract(lhs: UInt, _ rhs: UInt) -> (UInt, Bool)
  @transparent static func uncheckedMultiply(lhs: UInt, _ rhs: UInt) -> (UInt, Bool)
  @transparent static func uncheckedDivide(lhs: UInt, _ rhs: UInt) -> (UInt, Bool)
  @transparent static func uncheckedModulus(lhs: UInt, _ rhs: UInt) -> (UInt, Bool)
  @transparent func toUIntMax() -> UIntMax
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: UIntMax) -> UInt
}
extension UInt : BitwiseOperations {
  static var allZeros: UInt {
    @transparent get {}
  }
}
extension UInt {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: Int)
  @transparent func asSigned() -> Int
}
extension UInt {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension UInt : CVarArg {
  func encode() -> Word[]
}
```

### ``UInt16``
```scala
struct UInt16 : UnsignedInteger {
  var value: Int16
  @transparent init()
  @transparent init(_ v: Int16)
  @transparent init(_ value: UInt16)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> UInt16
  @transparent static func convertFromIntegerLiteral(value: UInt16) -> UInt16
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = UInt16
  func getArrayBoundValue() -> UInt16
  static var max: UInt16 {
    @transparent get {}
  }
  static var min: UInt16 {
    @transparent get {}
  }
}
extension UInt16 : Printable {
  var description: String {
    get {}
  }
}
extension UInt16 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension UInt16 : RandomAccessIndex {
  @transparent func succ() -> UInt16
  @transparent func pred() -> UInt16
  @transparent func distanceTo(other: UInt16) -> Int
  @transparent func advancedBy(amount: Int) -> UInt16
  @transparent static func uncheckedAdd(lhs: UInt16, _ rhs: UInt16) -> (UInt16, Bool)
  @transparent static func uncheckedSubtract(lhs: UInt16, _ rhs: UInt16) -> (UInt16, Bool)
  @transparent static func uncheckedMultiply(lhs: UInt16, _ rhs: UInt16) -> (UInt16, Bool)
  @transparent static func uncheckedDivide(lhs: UInt16, _ rhs: UInt16) -> (UInt16, Bool)
  @transparent static func uncheckedModulus(lhs: UInt16, _ rhs: UInt16) -> (UInt16, Bool)
  @transparent func toUIntMax() -> UIntMax
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: UIntMax) -> UInt16
}
extension UInt16 : BitwiseOperations {
  static var allZeros: UInt16 {
    @transparent get {}
  }
}
extension UInt16 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asSigned() -> Int16
}
extension UInt16 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension UInt16 : Reflectable {
  func getMirror() -> Mirror
}
extension CodeUnit : StringElement {
  static func toUTF16CodeUnit(x: CodeUnit) -> CodeUnit
  static func fromUTF16CodeUnit(utf16: CodeUnit) -> CodeUnit
}
extension UInt16 : CVarArg {
  func encode() -> Word[]
}
```

### ``UInt32``
```scala
struct UInt32 : UnsignedInteger {
  var value: Int32
  @transparent init()
  @transparent init(_ v: Int32)
  @transparent init(_ value: UInt32)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> UInt32
  @transparent static func convertFromIntegerLiteral(value: UInt32) -> UInt32
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = UInt32
  func getArrayBoundValue() -> UInt32
  static var max: UInt32 {
    @transparent get {}
  }
  static var min: UInt32 {
    @transparent get {}
  }
}
extension UInt32 : Printable {
  var description: String {
    get {}
  }
}
extension UInt32 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension UInt32 : RandomAccessIndex {
  @transparent func succ() -> UInt32
  @transparent func pred() -> UInt32
  @transparent func distanceTo(other: UInt32) -> Int
  @transparent func advancedBy(amount: Int) -> UInt32
  @transparent static func uncheckedAdd(lhs: UInt32, _ rhs: UInt32) -> (UInt32, Bool)
  @transparent static func uncheckedSubtract(lhs: UInt32, _ rhs: UInt32) -> (UInt32, Bool)
  @transparent static func uncheckedMultiply(lhs: UInt32, _ rhs: UInt32) -> (UInt32, Bool)
  @transparent static func uncheckedDivide(lhs: UInt32, _ rhs: UInt32) -> (UInt32, Bool)
  @transparent static func uncheckedModulus(lhs: UInt32, _ rhs: UInt32) -> (UInt32, Bool)
  @transparent func toUIntMax() -> UIntMax
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: UIntMax) -> UInt32
}
extension UInt32 : BitwiseOperations {
  static var allZeros: UInt32 {
    @transparent get {}
  }
}
extension UInt32 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asSigned() -> Int32
}
extension UInt32 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension UInt32 : Reflectable {
  func getMirror() -> Mirror
}
extension UInt32 {
  init(_ v: UnicodeScalar)
}
extension UInt32 : CVarArg {
  func encode() -> Word[]
}
```

### ``UInt64``
```scala
struct UInt64 : UnsignedInteger {
  var value: Int64
  @transparent init()
  @transparent init(_ v: Int64)
  @transparent init(_ value: UInt64)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> UInt64
  @transparent static func convertFromIntegerLiteral(value: UInt64) -> UInt64
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = UInt64
  func getArrayBoundValue() -> UInt64
  static var max: UInt64 {
    @transparent get {}
  }
  static var min: UInt64 {
    @transparent get {}
  }
}
extension UInt64 : Printable {
  var description: String {
    get {}
  }
}
extension UInt64 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension UInt64 : RandomAccessIndex {
  @transparent func succ() -> UInt64
  @transparent func pred() -> UInt64
  @transparent func distanceTo(other: UInt64) -> Int
  @transparent func advancedBy(amount: Int) -> UInt64
  @transparent static func uncheckedAdd(lhs: UInt64, _ rhs: UInt64) -> (UInt64, Bool)
  @transparent static func uncheckedSubtract(lhs: UInt64, _ rhs: UInt64) -> (UInt64, Bool)
  @transparent static func uncheckedMultiply(lhs: UInt64, _ rhs: UInt64) -> (UInt64, Bool)
  @transparent static func uncheckedDivide(lhs: UInt64, _ rhs: UInt64) -> (UInt64, Bool)
  @transparent static func uncheckedModulus(lhs: UInt64, _ rhs: UInt64) -> (UInt64, Bool)
  @transparent func toUIntMax() -> UIntMax
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: UIntMax) -> UInt64
}
extension UInt64 : BitwiseOperations {
  static var allZeros: UInt64 {
    @transparent get {}
  }
}
extension UInt64 {
  @transparent init(_ v: UInt8)
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asSigned() -> Int64
}
extension UInt64 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension UInt64 : Reflectable {
  func getMirror() -> Mirror
}
extension UInt64 {
  init(_ v: UnicodeScalar)
}
extension UInt64 : CVarArg {
  func encode() -> Word[]
}
```

### ``UInt8``
```scala
struct UInt8 : UnsignedInteger {
  var value: Int8
  @transparent init()
  @transparent init(_ v: Int8)
  @transparent init(_ value: UInt8)
  @transparent static func _convertFromBuiltinIntegerLiteral(value: Int2048) -> UInt8
  @transparent static func convertFromIntegerLiteral(value: UInt8) -> UInt8
  @transparent func _getBuiltinArrayBoundValue() -> Word
  typealias ArrayBoundType = UInt8
  func getArrayBoundValue() -> UInt8
  static var max: UInt8 {
    @transparent get {}
  }
  static var min: UInt8 {
    @transparent get {}
  }
}
extension UInt8 : Printable {
  var description: String {
    get {}
  }
}
extension UInt8 : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension UInt8 : RandomAccessIndex {
  @transparent func succ() -> UInt8
  @transparent func pred() -> UInt8
  @transparent func distanceTo(other: UInt8) -> Int
  @transparent func advancedBy(amount: Int) -> UInt8
  @transparent static func uncheckedAdd(lhs: UInt8, _ rhs: UInt8) -> (UInt8, Bool)
  @transparent static func uncheckedSubtract(lhs: UInt8, _ rhs: UInt8) -> (UInt8, Bool)
  @transparent static func uncheckedMultiply(lhs: UInt8, _ rhs: UInt8) -> (UInt8, Bool)
  @transparent static func uncheckedDivide(lhs: UInt8, _ rhs: UInt8) -> (UInt8, Bool)
  @transparent static func uncheckedModulus(lhs: UInt8, _ rhs: UInt8) -> (UInt8, Bool)
  @transparent func toUIntMax() -> UIntMax
  @transparent func toIntMax() -> IntMax
  @transparent static func from(x: UIntMax) -> UInt8
}
extension UInt8 : BitwiseOperations {
  static var allZeros: UInt8 {
    @transparent get {}
  }
}
extension UInt8 {
  @transparent init(_ v: Int8)
  @transparent init(_ v: UInt16)
  @transparent init(_ v: Int16)
  @transparent init(_ v: UInt32)
  @transparent init(_ v: Int32)
  @transparent init(_ v: UInt64)
  @transparent init(_ v: Int64)
  @transparent init(_ v: UInt)
  @transparent init(_ v: Int)
  @transparent func asSigned() -> Int8
}
extension UInt8 {
  @transparent init(_ v: Float)
  @transparent init(_ v: Double)
  @transparent init(_ v: Float80)
}
extension UInt8 : Reflectable {
  func getMirror() -> Mirror
}
extension CodeUnit : StringElement {
  static func toUTF16CodeUnit(x: CodeUnit) -> CodeUnit
  static func fromUTF16CodeUnit(utf16: CodeUnit) -> CodeUnit
}
extension UInt8 {
  init(_ v: UnicodeScalar)
}
extension UInt8 : CVarArg {
  func encode() -> Word[]
}
```

### ``UIntMax``
```scala
typealias UIntMax = UInt64
```

### ``UnicodeCodec``
```scala
protocol UnicodeCodec {
  typealias CodeUnit
  class func decode<G : Generator where `Self`.CodeUnit == CodeUnit>(inout next: G) -> UnicodeScalar?
  class func encode<S : Sink where `Self`.CodeUnit == CodeUnit>(input: UnicodeScalar, inout output: S)
}
```

### ``UnicodeScalar``
```scala
struct UnicodeScalar : ExtendedGraphemeClusterLiteralConvertible {
  var _value: Int32
  var value: UInt32 {
    get {}
  }
  static func convertFromExtendedGraphemeClusterLiteral(value: String) -> UnicodeScalar
  init()
  init(_ value: Int32)
  init(_ v: UInt32)
  init(_ v: UnicodeScalar)
  func escape() -> String
  func isASCII() -> Bool
  func isAlpha() -> Bool
  func isDigit() -> Bool
  var uppercase: UnicodeScalar {
    get {}
  }
  var lowercase: UnicodeScalar {
    get {}
  }
  func isSpace() -> Bool
}
extension UnicodeScalar : Streamable {
  func writeTo<Target : OutputStream>(inout target: Target)
}
extension UnicodeScalar : Printable {
  var description: String {
    get {}
  }
}
extension UnicodeScalar : Hashable {
  var hashValue: Int {
    get {}
  }
}
extension UnicodeScalar {
  init(_ v: Int)
}
extension UnicodeScalar : Comparable {
}
extension UnicodeScalar {
  func isPrint() -> Bool
}
```

### ``Unmanaged``
```scala
struct Unmanaged<T> {
  var _value: @sil_unmanaged T
  @transparent init(_private: T)
  @transparent static func fromOpaque(value: COpaquePointer) -> Unmanaged<T>
  @transparent func toOpaque() -> COpaquePointer
  @transparent static func passRetained(value: T) -> Unmanaged<T>
  @transparent static func passUnretained(value: T) -> Unmanaged<T>
  func takeUnretainedValue() -> T
  func takeRetainedValue() -> T
  @transparent func retain() -> Unmanaged<T>
  @transparent func release()
  @transparent func autorelease() -> Unmanaged<T>
}
```

### ``UnsafeArray``
```scala
struct UnsafeArray<T> : Collection, Generator {
  var startIndex: Int {
    get {}
  }
  var endIndex: Int {
    get {}
  }
  subscript (i: Int) -> T {
    get {}
  }
  init(start: UnsafePointer<T>, length: Int)
  func next() -> T?
  func generate() -> UnsafeArray<T>
  var _position: UnsafePointer<T>
  var _end: UnsafePointer<T>
}
```

### ``UnsafePointer``
```scala
struct UnsafePointer<T> : BidirectionalIndex, Comparable, Hashable, LogicValue {
  var value: RawPointer
  init()
  init(_ value: RawPointer)
  init(_ other: COpaquePointer)
  init(_ value: Int)
  init<U>(_ from: UnsafePointer<U>)
  static func null() -> UnsafePointer<T>
  static func alloc(num: Int) -> UnsafePointer<T>
  func dealloc(num: Int)
  var memory: T {
    @transparent get {}
    @transparent set {}
  }
  func initialize(newvalue: T)
  func move() -> T
  func moveInitializeBackwardFrom(source: UnsafePointer<T>, count: Int)
  func moveAssignFrom(source: UnsafePointer<T>, count: Int)
  func moveInitializeFrom(source: UnsafePointer<T>, count: Int)
  func initializeFrom(source: UnsafePointer<T>, count: Int)
  func initializeFrom<C : Collection where T == T>(source: C)
  func destroy()
  func destroy(count: Int)
  var _isNull: Bool {
    @transparent get {}
  }
  @transparent func getLogicValue() -> Bool
  subscript (i: Int) -> T {
    @transparent get {}
    @transparent set {}
  }
  var hashValue: Int {
    get {}
  }
  func succ() -> UnsafePointer<T>
  func pred() -> UnsafePointer<T>
  @conversion @transparent func __conversion() -> CMutablePointer<T>
  func __conversion() -> CMutableVoidPointer
  @conversion @transparent func __conversion() -> CConstPointer<T>
  @conversion @transparent func __conversion() -> CConstVoidPointer
  @conversion @transparent func __conversion() -> AutoreleasingUnsafePointer<T>
  init(_ cp: CConstPointer<T>)
  init(_ cm: CMutablePointer<T>)
  init(_ op: AutoreleasingUnsafePointer<T>)
  init(_ cp: CConstVoidPointer)
  init(_ cp: CMutableVoidPointer)
}
extension UnsafePointer<T> : Printable {
  var description: String {
    get {}
  }
}
```

### ``UnsignedInteger``
```scala
protocol UnsignedInteger : _UnsignedInteger, Integer {}
```

### ``UTF16``
```scala
struct UTF16 : UnicodeCodec {
  typealias CodeUnit = UInt16
  static func decode<G : Generator where CodeUnit == CodeUnit>(inout input: G) -> UnicodeScalar?
  static func encode<S : Sink where CodeUnit == CodeUnit>(input: UnicodeScalar, inout output: S)
  var _value
  init(_value: UInt16)
  init()
}
extension UTF16 {
  static func width(x: UnicodeScalar) -> Int
  static func leadSurrogate(x: UnicodeScalar) -> CodeUnit
  static func trailSurrogate(x: UnicodeScalar) -> CodeUnit
  static func copy<T : StringElement, U : StringElement>(source: UnsafePointer<T>, destination: UnsafePointer<U>, count: Int)
  static func measure<Encoding : UnicodeCodec, Input : Generator where Encoding.CodeUnit == Encoding.CodeUnit>(_: Encoding.Type, input: Input) -> (Int, Bool)
}
```

### ``UTF32``
```scala
struct UTF32 : UnicodeCodec {
  typealias CodeUnit = UInt32
  init(_ _value: UInt32)
  static func create(value: CodeUnit) -> UTF32
  func value() -> CodeUnit
  static func decode<G : Generator where CodeUnit == CodeUnit>(inout input: G) -> UnicodeScalar?
  static func encode<S : Sink where CodeUnit == CodeUnit>(input: UnicodeScalar, inout output: S)
  var _value
}
```

### ``UTF8``
```scala
struct UTF8 : UnicodeCodec {
  typealias CodeUnit = UInt8
  static func decode<G : Generator where CodeUnit == CodeUnit>(inout next: G) -> UnicodeScalar?
  static func encode<S : Sink where CodeUnit == CodeUnit>(input: UnicodeScalar, inout output: S)
  var _value
  init(_value: UInt8)
  init()
}
```

### ``UWord``
```scala
typealias UWord = UInt
```

### ``VaListBuilder``
```scala
@final class VaListBuilder {
  struct Header {
    var gp_offset
    var fp_offset
    var overflow_arg_area: UnsafePointer<Word>
    var reg_save_area: UnsafePointer<Word>
    init(gp_offset: UInt32, fp_offset: UInt32, overflow_arg_area: UnsafePointer<Word>, reg_save_area: UnsafePointer<Word>)
    init()
  }
  init()
  @final @final func append(arg: CVarArg)
  @final @final func va_list() -> CVaListPointer
  var gpRegistersUsed
  var sseRegistersUsed
  var header
  var storage: Word[]
  @objc deinit
}
```

### ``Void``
```scala
typealias Void = ()
```

### ``Word``
```scala
typealias Word = Int
```

### ``Zip2``
```scala
struct Zip2<S0 : Sequence, S1 : Sequence> : Sequence {
  typealias Stream1 = S0.GeneratorType
  typealias Stream2 = S1.GeneratorType
  typealias GeneratorType = ZipGenerator2<S0.GeneratorType, S1.GeneratorType>
  init(_ s0: S0, _ s1: S1)
  func generate() -> GeneratorType
  var sequences: (S0, S1)
}
```

### ``ZipGenerator2``
```scala
struct ZipGenerator2<E0 : Generator, E1 : Generator> : Generator {
  typealias Element = (E0.Element, E1.Element)
  init(_ e0: E0, _ e1: E1)
  func next() -> Element?
  var baseStreams: (E0, E1)
}
```

### ``C_ARGC: CInt``

### ``C_ARGV: UnsafePointer<CString>``

### ``__COLUMN__: Int``

Current column.

### ``__FILE__: String``

Current file name.

### ``__LINE__: Int``

Current line.


