---
layout: post
title: Grayscale with 2.13 inch E-Ink display
date: 2021-01-17 00:24 +0800
categories: blog
image: https://user-images.githubusercontent.com/72891/104817181-91b82f00-585a-11eb-8fcf-12de9374c72e.png
author: andelf
tags:
  - embedded
  - epd
  - rust
usemathjax: true
toc: true
published: false
---

[上回]({% post_url 2021-01-15-play-with-2-13-inch-e-ink-display %}) 说起淘了几个二手拆机墨水屏。

这回补剩下坑，说说如何在单色黑白屏幕上实现灰度图像显示。

## 墨水屏灰度显示

非黑即白？

上回驱动芯片 LUT 介绍里面，我们大概知道通过施加 15 个周期的 VSL/VSH 电压可以实现实现单像素从全黑到全白的转换，
从而实现墨水屏的局部刷新。每个周期是 $$T_{FRAME}$$.

然后留下一个问题，如果周期少于 15 呢？
测试发现，如果修改 LUT 字节，让周期小于 15 时，屏幕最终的显示结果就会没那么黑，差不多是浅灰或者深灰色。
而彻底不施加 VSL/VSH 电压，则在 VSS 下像素不会有任何变化。

这里开下脑洞, 如果每次刷新屏幕前更改 LUT, 设置不同的周期, 再加上局部刷新技术，是否可以实现同一屏幕多灰阶显示？
用脚趾头一想，大概有两种方案：

- 每次刷新一阶灰度，每次刷新前设置 LUT, 周期逐渐增加, 即先显示最浅像素，逐步显示深色像素（当然也可以反过来）
- 每次叠加最小一阶灰度，即使用最小周期的 LUT, 从最浅像素开始，每次把更深色的像素加深一些，叠加 15 次即得到最深色像素

测试发现，两种方法均可。但第一种因为屏幕素质不同的关系，会在灰阶过渡的边缘产生一条明显的线，
近距离观察比较明显。另外该方法刷新过程中色块逐渐从白色出现，不够自然。

这里以第二种方法介绍如何搞定灰阶显示。很容易想到，如果将 15 周期细分到不同程度，可以实现 2~4 bpp.
这里以 2bpp 为例，即一个象素有 4 级灰阶。

### 复习 LUT

上回说到, LUT 的局部刷新，大概是这么一个道理:

> `<<VS[0A-HH]:2/binary, VS[0A-HL]:2/binary, VS[0A-LH]:2/binary, VS[0A-LL]:2/binary>>``
>
> 先看 VS 部分，即驱动电压部分。`0x18 = 0b00_01_10_00`, 按照格式拆出:
>
> ```text
> VS[0A-HL] = 01
> VS[0A-LH] = 10
>
> # 其他情况为 00
> ```
>
> - 当像素点从白转黑时(HL), $$V_{pixel}$$ 电压为 VSH
> - 当象素点从黑转白时(LH), $$V_{pixel}$$ 电压为 VSL
> - 其他情况下，$$V_{pixel}$$ 电压为 VSS, 由芯片手册可知, VSS = VCOM, 相当于无变化
>
> (RP, TP 部分略)

那么针对“灰度逐阶叠加”这个需求，应该怎么搞？这里需要给出 HH, HL, LH, LL 的波形情况:

- 纯白像素，使用 HH, 不需要处理, 用 VSS
- 首次显示灰度，对应 HL, 需要加电压 VSH 显示黑色
- 加深灰阶，对应 LL, 即从黑到更黑，需要继续加电压 VSH 向更黑转
- 不加深灰阶，对应 LH, 不需要处理，用 VSS

所以 `VS[0A] = 0b00_01_00_01 = 0x11`. 即，只处理白转黑，和黑转更黑两种情况，其他情况均无变化。
整个 LUT 需要第一个 phase, 所以其他字节均为 0, 简单。

再看 RP, TP 部分:

> ```
>    // RP, repeat counter, 0 to 63, 0 means run time = 1
>    // TP, phase period, 0 to 31
>    // <<RP[0]_L:3/binary, TP[0A]:5/binary>>
>    // <<RP[0]_H:3/binary, TP[0B]:5/binary>>
>    0x0F, 0x01, // phase 0
>    ...
> ```

`RP = 0` 不需要变动, `TP[0A]` 此时按 2bpp 灰阶叠加需求减半得到 `TP[0A] = 0xF >> 1 = 0x7`.

### 灰度屏

当然，仔细查看市面上一些生产商给出的屏幕参数会发现，有屏幕本身就宣传是 4 阶灰度，即 2bpp.

查询部分对应的屏幕驱动芯片手册会发现，某些手册并没有直接介绍该芯片支持灰度，而是:

> 1 bit for white/black and 1 bit for red per pixel

不知看明白没，这样的芯片加屏幕组合，其实是利用了红色通道，实现了 2bpp 灰度。

另外例如 6 英寸 16 阶灰度屏幕，直接使用了 IT8951 这样的支持高阶灰度驱动芯片。芯片直接支持 4 bpp.

## 技术总结

ePaper 技术是最适合非持续更新应用场景。

墨水系统分类(ink system)

Peral/Carta 单色, 16 灰度
Spectra 三色
ACeP 全彩色, 单像素有 4 个粒子系统, 同出版业 CYMK 颜色标准
ref: https://www.eink.com/color-technology.html

## 三色屏幕

HINK-E0213A30-A0

212x104

SSD1675

70 字节 LUT

官方驱动黑白屏版本。

```c
const unsigned char LUT_DATA[] = {
    0x80, 0x60, 0x40, 0x00, 0x00, 0x00, 0x00, // LUT0: BB:     VS 0 ~7
    0x10, 0x60, 0x20, 0x00, 0x00, 0x00, 0x00, // LUT1: BW:     VS 0 ~7
    0x80, 0x60, 0x40, 0x00, 0x00, 0x00, 0x00, // LUT2: WB:     VS 0 ~7
    0x10, 0x60, 0x20, 0x00, 0x00, 0x00, 0x00, // LUT3: WW:     VS 0 ~7
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // LUT4: VCOM:   VS 0 ~7

    0x03, 0x03, 0x00, 0x00, 0x02, // TP0 A~D RP0
    0x09, 0x09, 0x00, 0x00, 0x02, // TP1 A~D RP1
    0x03, 0x03, 0x00, 0x00, 0x02, // TP2 A~D RP2
    0x00, 0x00, 0x00, 0x00, 0x00, // TP3 A~D RP3
    0x00, 0x00, 0x00, 0x00, 0x00, // TP4 A~D RP4
    0x00, 0x00, 0x00, 0x00, 0x00, // TP5 A~D RP5
    0x00, 0x00, 0x00, 0x00, 0x00, // TP6 A~D RP6

    0x15, 0x41, 0xA8, 0x32, 127, 0x03,
};
```

https://github.com/drewler/arduino-SSD1675A/blob/master/spi_demo.c 有三张 LUT 表，第一个和最后一个不错。

```
0x22, 0x11, 0x10, 0x00, 0x10, 0x00, 0x00,
0x11, 0x88, 0x80, 0x80, 0x80, 0x00, 0x00,
0x6a, 0x9b, 0x9b, 0x9b, 0x9b, 0x00, 0x00,
0x6a, 0x9b, 0x9b, 0x9b, 0x9b, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

0x04, 0x18, 0x04, 0x16, 0x01,
0x0a, 0x0a, 0x0a, 0x0a, 0x02,
0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00,
0x04, 0x04, 0x08, 0x3c, 0x07,
0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00'
```

注释中的 2in13 适合 (Creat by Yuhu Lin, 2014-04-16), 效果不错，时间长。

```
0xA5, 0x89, 0x10, 0x00, 0x00, 0x00, 0x00,
0xA5, 0x19, 0x80, 0x00, 0x00, 0x00, 0x00,
0xA5, 0xA9, 0x9B, 0x00, 0x00, 0x00, 0x00,
0xA5, 0xA9, 0x9B, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

0x0F, 0x0F, 0x0F, 0x0F, 0x02,
0x10, 0x10, 0x0A, 0x0A, 0x03,
0x08, 0x08, 0x09, 0x43, 0x07,
0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00,
```

Rust 项目自带:

```rust
const LUT: [u8; 70] = [
    // Phase 0     Phase 1     Phase 2     Phase 3     Phase 4     Phase 5     Phase 6
    // A B C D     A B C D     A B C D     A B C D     A B C D     A B C D     A B C D
    0b01001000, 0b10100000, 0b00010000, 0b00010000, 0b00010011, 0b00000000, 0b00000000,  // LUT0 - Black
    0b01001000, 0b10100000, 0b10000000, 0b00000000, 0b00000011, 0b00000000, 0b00000000,  // LUTT1 - White
    0b00000000, 0b00000000, 0b00000000, 0b00000000, 0b00000000, 0b00000000, 0b00000000,  // IGNORE
    0b01001000, 0b10100101, 0b00000000, 0b10111011, 0b00000000, 0b00000000, 0b00000000,  // LUT3 - Red
    0b00000000, 0b00000000, 0b00000000, 0b00000000, 0b00000000, 0b00000000, 0b00000000,  // LUT4 - VCOM

    // Duration            |  Repeat
    // A   B     C     D   |
    64,   12,   32,   12,    6,   // 0 Flash
    16,   8,    4,    4,     6,   // 1 clear
    4,    8,    8,    16,    16,  // 2 bring in the black
    2,    2,    2,    64,    32,  // 3 time for red
    2,    2,    2,    2,     2,   // 4 final black sharpen phase
    0,    0,    0,    0,     0,   // 5
    0,    0,    0,    0,     0    // 6
];
```

即:

```
0x48, 0xa0, 0x10, 0x10, 0x13, 0x00, 0x00,
0x48, 0xa0, 0x80, 0x00, 0x03, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x48, 0xa5, 0x00, 0xbb, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

0x40, 0x0c, 0x20, 0x0c, 0x06,
0x10, 0x08, 0x04, 0x04, 0x06,
0x04, 0x08, 0x08, 0x10, 0x10,
0x02, 0x02, 0x02, 0x40, 0x20,
0x02, 0x02, 0x02, 0x02, 0x02,
0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00,
```

无比耗时。

| 屏幕型号      | 屏幕排线编号      | 尺寸  | 分辨率  | 驱动 IC  | 描述               |
| :------------ | :---------------- | :---- | :------ | :------- | :----------------- |
|               | HINK-E0213A04-G01 | 2in13 | 250x122 | IL3895   | 黑白, 价签拆机     |
|               | HINK-E0213A30-A0  | 2in13 | 212x104 | SSD1675A | 红黑白, 价签拆机   |
| GDEH0213B72/3 | HINK-E0213A22-A0  | 2in13 | 250x122 | SSD1675B | 黑白, 官方 v2, EOL |
| GDEH0213Z19   | HINK-E0213A20-A2  | 2in13 | 212x104 | UC8151D  | 红黑白, 官方 b     |
|               | WFT0213CZ16       | 2in13 | 212×104 |          | 黄黑白, 官方 c     |
|               | WFT0213CZ16LW     | 2in13 | 212x104 |          | 黑白, 柔性, 官方 d |
| GDEH042Z96/21 | HINK-E042A13-A0   | 4in2  | 400x300 | SSD1619  | 红黑白, 价签拆机   |
|               |                   | 2in13 | 212x104 | IL0373F  | 红黑白             |
|               |                   | 1in54 | 200x200 | IL3829   | 黑白               |
|               |                   | 2in9  | 296x128 | IL3820   | 黑白               |

注 SSD1675A 和 SSD1675B 的 LUT 表长度不同。

Holitech(江西合力泰), screen from E-ink.

| FPC_label     | size  | resolution | IC                       | Description |
| ------------- | ----- | ---------- | ---                      | ----------- |
| HINK-E042A03  | 4in2  | 400x300    |                          | b/w         |
| HINK-E029A01  | 2in9  | 296x128    |                          | b/w         |
| HINK-E0213A01 | 2in13 | 250x122    |                          | b/w         |
| HINK-E0154A05 | 1in54 | 200x200    |                          | b/w         |
|               | 7in4  | 800x480    |                          | b/w/r       |
| HINK-E042A07  | 4in2  | 400x300    |                          | b/w/r       |
| HINK-E029A10  | 2in9  | 296x128    |                          | b/w/r       |
| HINK-E0213A07 | 2in13 | 212x104    |                          | b/w/r       |
| HINK-E0154A07 | 1in54 | 152x152    |                          | b/w/r       |
|               |       |            |                          |             |
| HINK-E0213A50 |       | 250x122    |                          | b/w         |
| HINK-E042A01  |       | 400×300    | SSD1608, SSD1618, SC5608 | 1bpp        |
|               |       |            |                          |             |

- ref: http://www.holitech.net/en/product/p12/index.html
- ref: https://www.holitech-europe.com/products/electronic-paper-displays
- ref: https://www.trs-star.com/en/products-en/displays/display-details/e-paper-displays
- ref: https://www.trs-star.com/project_files/dokumente/produktuebersicht/holitech_product_overview.pdf


上游：电子墨水屏膜量产厂家台湾元太 E-ink，市场份额高达 95%

中游：电子纸显示屏制造企业(因不好分辨，含销售)

- 东方科脉(DKE) -
- 合力泰(HINK)
- 无锡威峰(WF)
- 大连佳显，使用 WFT, HINK，不生产
- 清越光电(昆山维信诺科技有限公司已更名为苏州清越光电科技股份有限公司)
- 广州奥翼(OPM)
- 更多... (https://m.panelook.com/company_yp_list_cn.php?op=list&ac=supplier&topid=1&secid=73&subid=74)

| Name             | EN Name   | Symbol Prefix | URL                                             |
| :--------------- | :-------- | :------------ | :---------------------------------------------- |
| 东方科脉         | DKE       | DEPG          | https://china-epaper.com/                       |
| 合力泰           | Holitech  | HINK          | http://www.holitech.net/                        |
| 无锡威峰         | Weifeng   | WF            | http://www.wf-tech.com/                         |
| 盛辉电子（国际） | ThingWell | TINK          | http://www.lcdmaker.com/CN/E_paper_Display.html |
| 广州奥翼         | OED       | OPM           | http://www.oedtech.com/                         |

下游：ESL 系统解决方案提供商

- 汉朔科技 Hanshow
- 杭州智控
- 南京子午创力
- 杭州升腾智能 Suntown
- 深圳英伦科技
- 苏州汉朗光电 Halation
- 杭州中瑞思创

| Module   | Resolution | Driver IC |
| -------- | ---------- | --------- |
| DEPG0154 | 152x152    | SSD1680Z8 |
| DEPG0154 | 152x152    | UC8251D   |
| DEPG0154 | 152x152    | UC8251D   |
| DEPG0213 | 212x104    | SSD1680   |
| DEPG0213 | 212x104    | UC8251D   |
| DEPG0213 | 250x122    | SSD1680Z8 |

- ref: https://solisdisplay.com/dke-e-paper/
- ref: https://china-epaper.com/download/

| FPC_label | size  | resolution | IC  | Description |
| --------- | ----- | ---------- | --- | ----------- |
| E0213A22  | 2in13 | 250x122    |     | b/w/r       |
| E0213A50  | 2in13 | 250x122    |     | b/w         |
| E0154A44  | 1in54 | 200x200    |     | b/w/r       |
| E0154A45  | 1in54 | 200x200    |     | b/w         |
| E042A09   | 4in2  | 400x300    |     | b/w         |
| E043A13   | 4in2  | 400x300    |     | b/w/r       |
|           |       |            |     |             |

ref: http://www.einkcn.com/spec/
