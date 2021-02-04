---
layout: post
title: Raspberry Pi Pico Review
date: 2021-01-29 03:40 +0800
categories: blog
image: https://user-images.githubusercontent.com/72891/104817181-91b82f00-585a-11eb-8fcf-12de9374c72e.png
author: andelf
tags:
  - embedded
  - rp2040
  - rpi-pico
  - rust
usemathjax: true
toc: true
published: false
---


## PIO Instructions

3 位 opcode, 9 条指令, 其中 push/pull 共享 0b100. 位于高位, 用之后位区分.
`<<opcode:3, delay_and_side_set:5, oprand0:3, oprand1:5>>`.

delay, side_set

```
jmp
wait
in
out 
push
pull
mov
irq
set
```
