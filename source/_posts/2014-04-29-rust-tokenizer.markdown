---
layout: post
title: "Rust Tokenizer"
date: 2014-04-29 01:06:38 +0800
comments: true
categories: rust
---
A rust tokenizer from rust standard library.

```rust
extern crate syntax;

use syntax::parse;
use syntax::ast;

fn main() {
    let sess = parse::new_parse_sess();
    let cfg = Vec::new();
    let mut p = parse::new_parser_from_file(&sess, cfg, &Path::new("./mytest.rs"));
    while p.token != parse::token::EOF {
        p.bump();
        println!("debug => {}", parse::token::to_str(&p.token));
    }
}
```

DONE!

``p.parse_token_tree()`` will return a ``TokenTree``, which is a nested
token list.
