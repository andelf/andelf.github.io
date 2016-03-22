---
layout: post
title: "A rustdoc plugin"
date: 2014-04-29 02:26:48 +0800
comments: true
categories: rust
---

Run with

    rustdoc -L. --plugin-path . --plugins dummy rust-sdl2/src/sdl2/lib.rs

```rust
#![crate_id = "dummy#0.1"]
#![crate_type = "dylib"]


extern crate rustdoc;

use rustdoc::clean;

use rustdoc::plugins::{PluginCallback, PluginResult, PluginJson};

#[no_mangle]
pub fn rustdoc_plugin_entrypoint(c: clean::Crate) -> PluginResult {
    println!("loading extension ok!");
    println!("crate => {}", c.name);
    (c, None)
}
```

