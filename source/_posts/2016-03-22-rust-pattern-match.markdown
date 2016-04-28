---
layout: post
title: "Rust Pattern Match(Rust中的模式匹配)"
date: 2016-03-22 23:35:19 +0800
comments: true
categories: rust
---
# 模式匹配

汉语字典中对“模式”的解释是：事物的标准样式。在计算机科学中，它指特定类型的数据（往往是序列或是树形结构）满足某一特定结构或格式。“匹配”本身是指一个判断寻找过程。最早的模式匹配用于文本编辑器中的正则字符串搜索，之后才作为编程语言特性。

## 模式匹配基础

模式匹配在计算机科学领域有两层意思。其一，可以特指字符串匹配算法，例如为人熟知的 KMP 字符串匹配算法、命令行工具 grep 等。
其二，特指在一些语言中作为一种以结构的方式处理数据的工具，此时的匹配过程往往是树形匹配，与此相伴的往往还有一个特性叫 guard（守卫）。

Rust 中模式匹配随处可见，例如在``let``变量绑定语句、``match``匹配语句中等。利用好模式匹配这一特性可以使代码更简洁易懂。``Rust``支持模式匹配中的变量绑定、结构体/元组解构、守卫条件判断、数值范围匹配等特性。


### 原始匹配

``match`` 语句中可以直接匹配字面常量，下划线``_``匹配任意情形。

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

以上代码会打印出``one``。

### 结构匹配

``match`` 用于匹配一个表达式的值，寻找满足条件的子分支(``arm``)并执行。每个子分支包含三部分：一系列模式、可选的守卫条件以及主体代码块。

### 多个模式

每个子分支可以是多个模式，通过 ``|`` 符号分割：

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

以上代码打印出``one or two``。

### 守卫条件

通过``if``引入子分支的守卫条件：

```rust
enum OptionalInt {
    Value(i32),
    Missing,
}

let x = OptionalInt::Value(5);

match x {
    OptionalInt::Value(i) if i > 5 => println!("Got an int bigger than five!"),
    OptionalInt::Value(..) => println!("Got an int!"),
    OptionalInt::Missing => println!("No such luck."),
}
```

## 模式匹配进阶

其实进阶，不如直接从``libsyntax``源码看看到底模式匹配是如何实现。``syntax::ast::Pat``。

从AST源码中寻找语法要素屋外户两个要点，其一，语法要素是如何表达为对应AST的；其二，对应AST在哪些父AST中出现。

Rust中使用``syntax::ast::Pat``枚举来表示一个模式匹配。

```rust
pub struct Pat {
    pub id: NodeId,
    pub node: PatKind,
    pub span: Span,
}

pub enum PatKind {
    /// Represents a wildcard pattern (`_`)
    /// 表示通配，下划线
    Wild,

    /// A `PatKind::Ident` may either be a new bound variable,
    /// or a unit struct/variant pattern, or a const pattern (in the last two cases
    /// the third field must be `None`).
    ///
    /// In the unit or const pattern case, the parser can't determine
    /// which it is. The resolver determines this, and
    /// records this pattern's `NodeId` in an auxiliary
    /// set (of "PatIdents that refer to unit patterns or constants").
    Ident(BindingMode, SpannedIdent, Option<P<Pat>>),

    /// A struct or struct variant pattern, e.g. `Variant {x, y, ..}`.
    /// The `bool` is `true` in the presence of a `..`.
    Struct(Path, Vec<Spanned<FieldPat>>, bool),

    /// A tuple struct/variant pattern `Variant(x, y, z)`.
    /// "None" means a `Variant(..)` pattern where we don't bind the fields to names.
    TupleStruct(Path, Option<Vec<P<Pat>>>),

    /// A path pattern.
    /// Such pattern can be resolved to a unit struct/variant or a constant.
    Path(Path),

    /// An associated const named using the qualified path `<T>::CONST` or
    /// `<T as Trait>::CONST`. Associated consts from inherent impls can be
    /// referred to as simply `T::CONST`, in which case they will end up as
    /// PatKind::Path, and the resolver will have to sort that out.
    QPath(QSelf, Path),

    /// A tuple pattern `(a, b)`
    Tup(Vec<P<Pat>>),
    /// A `box` pattern
    Box(P<Pat>),
    /// A reference pattern, e.g. `&mut (a, b)`
    Ref(P<Pat>, Mutability),
    /// A literal
    Lit(P<Expr>),
    /// A range pattern, e.g. `1...2`
    Range(P<Expr>, P<Expr>),
    /// `[a, b, ..i, y, z]` is represented as:
    ///     `PatKind::Vec(box [a, b], Some(i), box [y, z])`
    Vec(Vec<P<Pat>>, Option<P<Pat>>, Vec<P<Pat>>),
    /// A macro pattern; pre-expansion
    Mac(Mac),
}
```

以上AST定义，即说明，到底什么被认为是一个“模式”。

以下介绍``Pat``在哪些AST中出现。

### 全局 Item

全局 Item 中，使用模式匹配的均为函数参数。

#### ItemKind::Fn 

``Fn`` 全局函数 -> ``FnDecl`` 函数声明 -> ``[Arg]`` 函数头参数声明。

#### ItemKind::Trait

``Trait`` -> ``[TraitItem]`` -> ``TraitItemKind::Method`` -> ``MethodSig`` -> ``FnDecl`` 方法声明，同上。

#### ItemKind::Impl 

``Impl`` -> ``[ImplItem]`` -> ``ImplItemKind::Method`` -> ``MethodSig`` -> ``FnDecl``。

### ast::Stmt 语句

#### StmtKind::Decl

``Decl`` -> ``DeclKind::Local``。

即 ``let`` 语句 ``let <pat>:<ty> = <expr>;``。

#### StmtKind::Expr 表达式

见下。

### ast::Expr

除``match``外，``if let``、``while let``、``for``控制语句支持同时进行模式匹配。具体实现是一种``desugared``过程，即，去语法糖化。

同时类似于函数定义，闭包参数也支持模式匹配。

#### if let

``IfLet(P<Pat>, P<Expr>, P<Block>, Option<P<Expr>>)``

``if let pat = expr { block } else { expr }``

This is desugared to a match expression.

#### while let

``WhileLet(P<Pat>, P<Expr>, P<Block>, Option<Ident>)``

``'label: while let pat = expr { block }``

#### for

``ForLoop(P<Pat>, P<Expr>, P<Block>, Option<Ident>)``

``'label: for pat in expr { block }``

#### match 

``Match(P<Expr>, Vec<Arm>)``

``match`` 语句，在 ``Arm`` 中出现，其中 ``Arm`` 定义为

```
pub struct Arm {
    pub attrs: Vec<Attribute>,
    pub pats: Vec<P<Pat>>,
    pub guard: Option<P<Expr>>,
    pub body: P<Expr>,
}
```

#### 闭包

``Closure(CaptureBy, P<FnDecl>, P<Block>)``

闭包，例如 ``move |a, b, c| {a + b + c}``。


## 相关 feature gate

``advanced_slice_patterns`` - See the match expressions section for discussion; the exact semantics of slice patterns are subject to change, so some types are still unstable.

``slice_patterns`` - OK, actually, slice patterns are just scary and completely unstable.

``box_patterns`` - Allows box patterns, the exact semantics of which is subject to change.


## 参考

https://doc.rust-lang.org/book/patterns.html
