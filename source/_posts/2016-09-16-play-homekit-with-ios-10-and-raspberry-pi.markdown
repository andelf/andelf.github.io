---
layout: post
title: "折腾 Raspberry Pi + HomeKit 手记"
date: 2016-09-16 20:34:43 +0800
comments: true
categories: ios
---

9月14日凌晨苹果终于推送了 iOS 10 的更新。从之前发布会来看，并没有多少亮点，除了几天的新鲜感之外，
尤其是对于目前还在用上两代机型的我来说，2333。

两年前苹果发布 Swift 语言的同时，新增了 HomeKit，当时用工具 dump 过最老版本的 Swift 声明。传送门：[HomeKit.swift](https://github.com/andelf/Defines-Swift/blob/6a8cda2e12bf6e5a23979a1ad121e70a0eeef6dd/HomeKit.swift)。目前所有官方相关的资料位于 [HomeKit - Apple](http://www.apple.com/ios/home/)。

好消息是期待很久的 HomeKit 应用终于上线，屏幕上多了“家庭(Home)”应用，
iOS 10 终于强化了推出已有两年智能家居平台，提供了官方 App，有不少硬件厂商支持。

简单说，HomeKit 就是苹果官方的智能家居平台解决方案，包括移动设备 SDK，智能家居硬件通信协议(HAP: HomeKit Accessory Protocol)、以及 MFi(Made for iPhone/iPod/iPad) 认证等等。通过 WiFi 或蓝牙连接智能家居设备（或 bridge 设备），也可以利用 Apple TV(4代) 或闲家中的置 iPad 实现设备的远程控制（HAP over iCloud）。

众所周知苹果是卖数据线等硬件的公司（嗯，假设你数据线也坏过不少），HAP 协议部分是需要加入 MFi Program 才能获取文档，而且 MFi Program 无法以个人开发者身份加入。

好在有好心人逆向了 HAP 的服务端协议（对于智能硬件来说，硬件是服务端，手机App是客户端）。

对于折腾党来说，机会来了，自己动手改造家居！本文不涉及 App 开发，只涉及如何自制支持 HomeKit 的设备。

## 准备工作

设备列表：

- iPhone 6P (iOS 10)
- Raspberry Pi 3 (Debian jessie)

考察了两个比较靠谱的 HAP 实现：

- https://github.com/KhaosT/HAP-NodeJS
- https://github.com/brutella/hc (golang)

最终选择使用 golang 的 ``brutella/hc``，准备环境。

需要保证树莓派和手机位于统一子网，因为 HAP 底层是基于 Apple mDNS(RFC 6762)。

``brutella/hc`` 要求 golang >= 1.4，而 Debian jessie 版本较低，配置 jessie-backports 即可。

安装好 golang 1.6.2，建立开发目录。

## 示例

跑通官方示例代码：

```golang
package main

import (
	"github.com/brutella/hc"
	"github.com/brutella/hc/accessory"
	"log"
)

func main() {
	info := accessory.Info{
		Name:         "Lamp",
		SerialNumber: "051AC-23AAM1",
		Manufacturer: "Apple",
		Model:        "AB",
	}
	acc := accessory.NewSwitch(info)

	acc.Switch.On.OnValueRemoteUpdate(func(on bool) {
		if on == true {
			log.Println("Client changed switch to on")
		} else {
			log.Println("Client changed switch to off")
		}
	})

	config := hc.Config{Pin: "00102003"}
	t, err := hc.NewIPTransport(config, acc.Accessory)
	if err != nil {
		log.Fatal(err)
	}

	hc.OnTermination(func() {
		t.Stop()
	})

	t.Start()
}
```

编译执行.

```
$ AppleHome> # current dir

$ AppleHome> go get
...

$ AppleHome> go build
...

$ AppleHome> ./AppleHome
...
```

随后打开手机的 Home App，添加设备，选择 Lamp，输入 PIN 00102003，完成配对，即可使用。


