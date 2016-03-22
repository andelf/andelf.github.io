---
layout: post
title: "Swift Standard Library (Swift 标准库)"
date: 2014-06-06 10:49:26 +0800
comments: true
categories: swift
---

所有默认函数在 Swift 名字空间，即 Swift.abs 这样。

所有的函数功能都是我猜的。猜错别找我。

## 第一部分 a to z

### ``abs``
```
abs(x: T) -> T
```

绝对值， 所有数字类型均可。

    abs(-20.3)
    
### ``advance``

```
advance(start: T, n: T.DistanceType) -> T
advance(start: T, n: T.DistanceType, end: T) -> T
```

计算步进和距离的函数貌似。

    advance(0, 20) // 20
    advance(2, 20, 4) // 4

### ``alignof alignofValue``

```
alignof(T.Type) -> Int
alignofValue(T) -> Int    
```

计算类型的 alignment。    

```
alignof(Int)
alignof(Uint8)
alignofValue(Float(3.2))    
```

### ``assert``

```
assert(condition: @auto_closure () -> Bool, message: StaticString, file: StaticString, line: UWord) -> Void
assert(condition: @auto_closure () -> T, message: StaticString, file: StaticString, line: UWord) -> Void
```

断言语句。因为是 @auto_closure, 所以直接写语句就可以了，第二个 T 的限制是 LogicValue, 所以 Optional 也可以。

```
let age = 17
assert(age > 0, "Alive Only")
assert(age > 18, "Adult Only", file: __FILE__, line: __LINE__)
assert("342345".toInt())
```

### ``bridge*``
```
bridgeFromObjectiveC(x: AnyObject, T.Type) -> T?
bridgeFromObjectiveCUnconditional(x: AnyObject, T.Type) -> T
bridgeToObjectiveC(x: T) -> AnyObject?
bridgeToObjectiveCUnconditional(x: T) -> AnyObject
```

### ``c_malloc_size``

    c_malloc_size(heapMemory: UnsafePointer<Void>) -> Int
    
### ``c_memcpy``

    c_memcpy(dest: UnsafePointer<Void>, src: UnsafePointer<Void>, size: UInt) -> Void
    
### ``c_putchar(value: Int32) -> Void``

    c_putchar(value: Int32) -> Void
    
就是 putchar，最基础的 output 语句。
    
    c_putchar(97) // ASCII 'a'
    
### ``contains``

```
contains(seq: S, predicate: (S.GeneratorType.Element) -> L) -> Bool
contains(seq: S, x: S.GeneratorType.Element) -> Bool
```

判断序列是否包含某元素、是否包含满足某条件的元素。
    
```
let a = [1,2,4,56,7]
contains(a, { $0 > 50 }) // true
contains(a, 56)
```

### ``count countElements``

```
count(r: Range<I>) -> I.DistanceType
countElements(x: T) -> T.IndexType.DistanceType
```
    
数元素、Range 个数。    
    
```
count(1..20)
let a = [1,2,4,56,7]
countElements(a) // 5
```

### ``countLeadingZeros``

    countLeadingZeros(value: Int64) -> Int64
    
Count Leading Zeros in Binary Format.
二进制格式前面有多少个 0.

```
countLeadingZeros(1) // 63
countLeadingZeros(-1) // 0
```

### ``debugPrint debugPrintln``

```
debugPrint(object: T) -> Void
debugPrint(object: T, &target: TargetStream) -> Void
debugPrintln(object: T) -> Void
debugPrintln(object: T, &target: TargetStream) -> Void
```

打印任意对象。``TargetStream`` 暂时未知。

### ``distance``

    distance(start: T, end: T) -> T.DistanceType

起止距离.

    distance(1, 20) // 19

### ``dropFirst dropLast``

```
dropFirst(seq: Seq) -> Seq.SliceType
dropLast(seq: Seq) -> Seq.SliceType
```

序列操作函数。

```
let b = [1,2,3,4,5]
// b : Array<Int> = [1, 2, 3, 4, 5]
dropFirst(b)
// r16 : Slice<Int> = [2, 3, 4, 5]
dropLast(b)
// r17 : Slice<Int> = [1, 2, 3, 4]
```

### ``dump``
```
dump(x: T, name: String?, indent: Int, maxDepth: Int, maxItems: Int) -> T
dump(x: T, name: String?, indent: Int, maxDepth: Int, maxItems: Int, &targetStream: TargetStream) -> T
```

打印变量的详情。

```
let b = [1,2,3,4,5]
dump(b) // detailed output at console
dump(b, name: "A Array B")
dump("Hello World")
```

### ``encodeBitsAsWords``

    encodeBitsAsWords(x: T) -> Word[]
    
未知作用。尝试发现类型是 ASCII 码。    
    
    encodeBitsAsWords(97) // [97]: Word[]
    
### ``enumerate``

    enumerate(seq: Seq) -> EnumerateGenerator<Seq.GeneratorType>

同 Python enumerate.

```
let b = [1,2,3,4,5]
for (idx, val) in enumerate(b) { println("b \(idx) = \(val)") }
```

### ``equal``

```
equal(a1: S1, a2: S2) -> Bool
equal(a1: S1, a2: S2, pred: (S1.GeneratorType.Element, S1.GeneratorType.Element) -> Bool) -> Bool
```

比较相等。

```
equal(2, 1+1)
equal([1,2,3,4], [2,4,6,8], { $0 * 2 == $1 })
```

### ``fatalError``

    fatalError(message: StaticString, file: StaticString, line: UWord) -> Void

Runtime fatal error. then quit.
    
    fatalError("Taylor Swift Sucks")

### ``filter``

```
filter(source: C, includeElement: (C.GeneratorType.Element) -> Bool) -> FilterCollectionView<C>
filter(source: S, includeElement: (S.GeneratorType.Element) -> Bool) -> FilterSequenceView<S>
```

同 Python filter。

```
let b = [1,2,3,4,5]
for i in filter(b, { $0 > 3 }) { println("got \(i)") }
for i in filter(1...100, { $0 % 17 == 0 }) { println("got \(i)") }
```

### ``find``

    find(domain: C, value: C.GeneratorType.Element) -> C.IndexType?
    
查找某一元素的索引值。

```
let b = [1,2,3,4,5]
find(b, 20) // nil
find(b, 2)  // 1
```

### ``getBridgedObjectiveCType``

    getBridgedObjectiveCType(T.Type) -> Any.Type?
    
### ``getVaList``

    getVaList(args: CVarArg[]) -> CVaListPointer
    
### ``indices``

    indices(seq: Seq) -> Range<Seq.IndexType>

索引的迭代器。

```
let b = [1,2,3,4,5]
for i in indices(b) { println(i) }
// 0 1 2 3 4
```

### ``insertionSort``

```
insertionSort(&elements: C, range: Range<C.IndexType>) -> Void
insertionSort(&elements: C, range: Range<C.IndexType>, &less: (C.GeneratorType.Element, C.GeneratorType.Element) -> Bool) -> Void
```

插入排序。range 表示排序范围。

```
var tt = [23, 34, 56, 45, 67, 12, 78, 89, 90]
insertionSort(&tt, 0..5)
println(tt)
```

### ``isBridgedToObjectiveC isBridgedVerbatimToObjectiveC``

```
isBridgedToObjectiveC(T.Type) -> Bool
isBridgedVerbatimToObjectiveC(T.Type) -> Bool
```

### ``isUniquelyReferenced``

    isUniquelyReferenced(&x: T) -> Bool    
    
### ``join``

    func join<C : ExtensibleCollection, S : Sequence where C == C>(separator: C, elements: S) -> C

序列的 join 操作

    join("!!", ["Hello", "World"])
   
### ``lexicographicalCompare``

```
lexicographicalCompare(a1: S1, a2: S2) -> Bool
lexicographicalCompare(a1: S1, a2: S2, less: (S1.GeneratorType.Element, S1.GeneratorType.Element) -> Bool) -> Bool
```

### ``map``

```
map(source: C, transform: (C.GeneratorType.Element) -> T) -> MapCollectionView<C, T>
map(source: S, transform: (S.GeneratorType.Element) -> T) -> MapSequenceView<S, T>
map(x: T?, f: (T) -> U) -> U?
```

同 Python map 操作。

```
for i in map([1,2,3,4], { $0 * 10 }) { println(i) }
let optVal: Int? = 250
let v: Int? =  map(optVal, { $0 + 1 }) // 251
```

### ``max maxElement``

```
max(x: T, y: T, rest: T[]) -> T
maxElement(range: R) -> R.GeneratorType.Element
```
    
最大值。    
    
```
max(2, 4)
max(1,2,4,5,6,100,4,5,6,7) // 100
maxElement([1,2,4,5,6,100,4,5,6,7])
```

### ``min minElement``

```
min(x: T, y: T, rest: T[]) -> T
minElement(range: R) -> R.GeneratorType.Element
```

最小值。Same usage as ``max``

### ``numericCast``

    numericCast(x: T) -> U

Cast between unsigned and signed integers.

```
let bb: UInt8 = 20
let cc: Int = numericCast(bb)
```

### ``partition``

```
partition(&elements: C, range: Range<C.IndexType>) -> C.IndexType
partition(&elements: C, range: Range<C.IndexType>, &less: (C.GeneratorType.Element, C.GeneratorType.Element) -> Bool) -> C.IndexType
``` 

Unknown usage.

```
var tt = [12, 23, 34, 45, 56, 67, 78, 89, 90]
println(partition(&tt, 3..4))
```

### ``posix_read posix_write``

```
posix_read(fd: Int32, buf: RawPointer, sz: Int) -> Int
posix_write(fd: Int32, buf: RawPointer, sz: Int) -> Int
```

### ``print println``

```
print(object: T) -> Void
print(object: T, &target: TargetStream) -> Void
println() -> Void
println(object: T) -> Void
println(object: T, &target: TargetStream) -> Void
```

打印语句，们。

### ``quickSort``

```
quickSort(&elements: C, range: Range<C.IndexType>) -> Void
quickSort(&elements: C, range: Range<C.IndexType>, less: (C.GeneratorType.Element, C.GeneratorType.Element) -> Bool) -> Void
```

快排序, range表示需要排序的范围。

```
var tt = [23, 34, 56, 45, 67, 12, 78, 89, 90]
quickSort(&tt, 0..5)
quickSort(&tt, 0..5, { $1 < $0 }) // 逆序
``` 

### ``reduce``

    reduce(sequence: S, initial: U, combine: (U, S.GeneratorType.Element) -> U) -> U
    
同 Python 中 reduce, 类似 Haskell fold 系列函数。

```
// reduce 应用之 求和
let tt = [23, 34, 56, 45, 67, 12, 78, 89, 90]
reduce(tt, 0, { $0 + $1 })
```

### ``reflect ``

    reflect(x: T) -> Mirror
    
暂时未知

### ``reinterpretCast``

    reinterpretCast(x: T) -> U

相同 size 的类型，之间的直接 cast.

```
let a: Int = -1
let c: UInt = reinterpretCast(a)
println(c)
println(UInt.max) // 一定等于 c 。别问我为什么。
```

### ``reverse``

    reverse(source: C) -> ReverseView<C>
   
同 Python reversed()，逆序的迭代器。

```
let tt = [23, 34, 56, 45, 67, 12, 78, 89, 90]
for i in reverse(tt) { println(i) }
```
    
### ``roundUpToAlignment``

    roundUpToAlignment(offset: Int, alignment: Int) -> Int
    
内存 alignment 计算函数。比如 alignment 为 4 字节，offset 为 5, 那么将在下个 4 字节， 即 8 的位置上对齐。

```
roundUpToAlignment(2, 4) // 4
roundUpToAlignment(5, 4) // 8
```

### ``sizeof sizeofValue``

```
sizeof(T.Type) -> Int
sizeofValue(T) -> Int
```

计算类型或者值的 size。

```
sizeof(Int)
sizeofValue(2.4)  // a Double
```

### ``sort``

```
sort(array: T[]) -> T[]
sort(array: T[], pred: (T, T) -> Bool) -> T[]
```

排序函数。类似 Python sorted()。

```
let foo = [18, 14, 21, 8, 45, 12, 9]
sort(foo)
sort(foo, { String($0) < String($1) }) // 按照 ASCII 顺序排序
```

### ``split``

    split(seq: Seq, isSeparator: (Seq.GeneratorType.Element) -> R, maxSplit: Int, allowEmptySlices: Bool) -> Seq.SliceType[]
    
分割序列函数。
    
```
split("Hello World", { $0 == " " })
// : String[] = ["Hello", "World"]
```

### ``startsWith``
    
    startsWith(s0: S0, s1: S1) -> Bool
    
判断序列开头子序列。    
    
```
startsWith(1..100, 1..2)
startsWith("Hello World", "Hel")
```

### ``strideof strideofValue``

```
strideof(T.Type) -> Int
strideofValue(T) -> Int
```

计算内存 stride 。用法同 sizeof sizeofValue。

### ``swap``

    swap(&a: T, &b: T) -> Void
   
交换值。

```
var a = 1, b = 2
swap(&a, &b)
```

### ``swift_*``

```
swift_bufferAllocate(bufferType: HeapBufferStorageBase.Type, size: Int, alignMask: Int) -> AnyObject
swift_keepAlive(&T) -> Void
swift_MagicMirrorData_summaryImpl(metadata: Any.Type, result: UnsafePointer<String>) -> Void
```

### ``toString``

    toString(object: T) -> String
    
这个没啥好说的。就是 toString。例子我都懒得举。
    
### ``transcode``

```
transcode(inputEncoding: Encoding.Type, outputEncoding: Encoding.Type, input: Input, output: Output) -> Void
transcode(inputEncoding: InputEncoding.Type, outputEncoding: OutputEncoding.Type, input: Input, output: Output) -> Void
```

未知。

### ``underestimateCount``

    underestimateCount(x: T) -> Int
    
预估 Sequence 的成员个数。

```
underestimateCount(1..1000)
```
    
### ``unsafeReflect    ``
    
    unsafeReflect(owner: NativeObject, ptr: UnsafePointer<T>) -> Mirror

未知。

### ``withExtendedLifetime``

    withExtendedLifetime(x: T, f: (T) -> Result) -> Result

### ``withObjectAtPlusZero``

    withObjectAtPlusZero(x: AnyObject, f: (COpaquePointer) -> Result) -> Result
    
### ``withUnsafePointer withUnsafePointers withUnsafePointerToObject``

```
withUnsafePointer(&arg: T, body: (UnsafePointer<T>) -> Result) -> Result
withUnsafePointers(&arg0: A0, &arg1: A1, &arg2: A2, body: (UnsafePointer<A0>, UnsafePointer<A1>, UnsafePointer<A2>) -> Result) -> Result
withUnsafePointers(&arg0: A0, &arg1: A1, body: (UnsafePointer<A0>, UnsafePointer<A1>) -> Result) -> Result
withUnsafePointerToObject(&arg: T?, body: (UnsafePointer<ImplicitlyUnwrappedOptional<T>>) -> Result) -> Result
``` 

来自 Rust 的同学对这里一定不会太陌生。不过作用和使用场合未知。

### ``withVaList``

```
withVaList(args: CVarArg[], f: (CVaListPointer) -> R) -> R
withVaList(builder: VaListBuilder, f: (CVaListPointer) -> R) -> R
```

