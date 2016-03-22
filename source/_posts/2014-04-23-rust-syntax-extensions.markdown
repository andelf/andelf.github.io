---
layout: post
title: "Rust Syntax Extensions"
date: 2014-04-23 23:44:43 +0800
comments: true
categories: rust
---

All rust ASTs are in ``syntax::ast``. And extensions are in
``syntax::ext``.

```rust
#![feature(macro_registrar, managed_boxes)];

extern crate syntax;

use syntax::ast;
use syntax::ast::Name;
use syntax::ext::base::SyntaxExtension;

#[macro_registrar]
pub fn macro_registrar(register: |Name, SyntaxExtension|);
```

Managed boxes, the ``@-pointer`` is still used in rust compiler, so it
is essential to enable this feature.

where the doc says:

```rust
/// A name is a part of an identifier, representing a string or
gensym. It's the result of interning.
/// created by token::intern()
type Name = u32;

/// An enum representing the different kinds of syntax extensions.
pub enum SyntaxExtension {
    /// A syntax extension that is attached to an item and creates new items
    /// based upon it.
    ///
    /// `#[deriving(...)]` is an `ItemDecorator`.
    ItemDecorator(ItemDecorator),

    /// A syntax extension that is attached to an item and modifies it
    /// in-place.
    ItemModifier(ItemModifier),

    /// A normal, function-like syntax extension.
    ///
    /// `bytes!` is a `NormalTT`.
    NormalTT(~MacroExpander:'static, Option<Span>),

    /// A function-like syntax extension that has an extra ident before
    /// the block.
    ///
    /// `macro_rules!` is an `IdentTT`.
    IdentTT(~IdentMacroExpander:'static, Option<Span>),
}

type ItemDecorator = fn(&mut ExtCtxt, Span, @MetaItem, @Item, |@Item|);
type ItemModifier = fn(&mut ExtCtxt, Span, @MetaItem, @Item) -> @Item;

pub trait MacroExpander {
    fn expand(&self, ecx: &mut ExtCtxt, span: Span, token_tree: &[TokenTree]) -> ~MacResult;
}

pub trait IdentMacroExpander {
    fn expand(&self, cx: &mut ExtCtxt, sp: Span, ident: Ident, token_tree: Vec<TokenTree>) -> ~MacResult;
}

/// The result of a macro expansion. The return values of the various
/// methods are spliced into the AST at the callsite of the macro (or
/// just into the compiler's internal macro table, for `make_def`).
pub trait MacResult {
    /// Define a new macro.
    fn make_def(&self) -> Option<MacroDef> {
        None
    }
    /// Create an expression.
    fn make_expr(&self) -> Option<@ast::Expr> {
        None
    }
    /// Create zero or more items.
    fn make_items(&self) -> Option<SmallVector<@ast::Item>> {
        None
    }

    /// Create a statement.
    ///
    /// By default this attempts to create an expression statement,
    /// returning None if that fails.
    fn make_stmt(&self) -> Option<@ast::Stmt> {
        self.make_expr()
            .map(|e| @codemap::respan(e.span, ast::StmtExpr(e, ast::DUMMY_NODE_ID)))
    }
}
```

## How to call

```rust
use syntax::parse::token;

#[macro_registrar]
pub fn registrar(register: |Name, SyntaxExtension|) {
    register(token::intern("your_ext_name"), your_extension);
}
```

Note: ``registrar`` is not ``register``.

How to create each part:

``Name`` is generated from
``syntax::parse::token::intern()``.

For different SyntaxExtension Enum, write the function, and wrap in Enum constructor.

### ItemDecorator

``src/libsyntax/ext/deriving/mod.rs``. The
``#[deriving(...)``.

To make items from an existing item.

### ItemModifier

``src/test/auxiliary/macro_crate_test.rs``

Modify existing item.

### NormalTT

Builtin sample in: ``src/libsyntax/ext/base.rs`` and sub dirs.
Normal ``macro_name!(...)`` call.

### IdentTT

Builtin sample in: ``src/libsyntax/ext/tt/macro_rules.rs``, ``pub fn add_new_extension``.

```
macro_name! ident(
    whatever here...
)
```

## Aux

The ``quote`` feature.

```
#![feature(quote, phase)]
```

This enables ``quote_expr!(ExtCtxt, [Code])``, ``quote_tokens!``,
``quote_item!``, ``quote_pat!``, ``quote_stmt!``, ``quote_ty!`` macros,
and generates corresponding ast type.
