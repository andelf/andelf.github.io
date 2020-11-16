---
layout: post
title: "广州实时工具App逆向"
date: 2015-06-18 17:39:06 +0800
comments: true
categories: blog
tags:
- crack
---

简记。用了 IDA Pro，安卓手机的 Remote 客户端。以及 apktool 等。

[Github: guangzhou-realtime-bus](https://github.com/andelf/guangzhou-realtime-bus)

- 生成 e=3 的 1024 位 RSA 密钥对
- 公钥串用查表加密(byte 映射)，然后 base64 封装发送给服务器
- 服务器返回一串用公钥加密过的数据
- 用本地私钥解密后，该数据包含未知96字节的一段数据和 DES Key
- 从此通信用 DES 加密

base64封装过程：先打包字符串长度，然后是原始字符串（JSON），然后是``0x10``(md5字符串长度)，
然后是 md5 校验值。整个二进制字符串用 base64 转码，POST 给服务器。

具体的登录注册过程还需要进一步抓包分析，不过暂时兴趣不在这里了。
