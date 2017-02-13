---
layout: post
title: "在 CircleCI 上使用 Rust(CircleCI meets Rust)"
date: 2016-11-18 13:26:15 +0800
comments: true
categories: rust ci
---
最近由于频频遇到 travis-ci 的问题，主要是 Linux 资源排队、macOS 资源更需要排队，导致自动测试时间被拉长，
影响开发效率。

了解到 CircleCI 是不错的替代品，所以打算迁移 Rust 项目过去。当然说起来， CircleCI 的野心更大，是要来替代 jenkins 的。

目前官方支持语言其实都比较落后，包括 go 也只是 1.6 版本，但似乎不是问题，而且据介绍， CircleCI 2.0 支持自定义 build image，支持语言的版本当然不在话下。

每天面对各种 IaaS, PaaS，免不了写配置是，这也是 yaml 程序员的日常。

```yaml
dependencies:
  pre:
    - curl https://sh.rustup.rs -sSf | sh
    
test:
  override:
    - cargo build
    - cargo test
```

如上。然而不 work。报错：

```
cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
warning: spurious network error (2 tries remaining): [12/-12] Malformed URL 'ssh://git@github.com:/rust-lang/crates.io-index'
warning: spurious network error (1 tries remaining): [12/-12] Malformed URL 'ssh://git@github.com:/rust-lang/crates.io-index'
error: failed to fetch `https://github.com/rust-lang/crates.io-index`

To learn more, run the command again with --verbose.

cargo build returned exit code 101

Action failed: cargo build
```

神了。原来， CircleCI 自作聪明在 ``.gitconfig`` 里修改了映射配置，强制用它自己的 ssh key 去访问 github，rewrite 了 ``https://github.com`` 的所有仓库。
这恰恰和 cargo 的 registry 机制冲突。所以报错。

> CircleCI has rewrite ``https:://github.com`` to ``ssh://git@github.com:`` in ``.gitconfig``. And this made cargo fail with above error message.

找到了原因，就可以搞了：

```yaml
machine:
  pre:
    - sed -i 's/github/git-non-exist-hub/g' ~/.gitconfig
    
dependencies:
  pre:
    - curl https://sh.rustup.rs -sSf | sh
    
test:
  override:
    - cargo build
    - cargo test
```

嗯, Ugly but works.



