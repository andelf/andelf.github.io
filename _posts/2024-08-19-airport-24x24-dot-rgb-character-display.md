---
layout: post
title: 机场 24x24 像素显示单元 Airport 24x24 Dot Matrix RGB Character Display Unit
date: 2024-08-19 21:42 +0800
published: true
author: andelf
image: /assets/display-24x24-final.jpeg
tags:
  - embedded
  - rust
usemathjax: true
toc: true
---

之前在咸鱼 App 上收了 3 个 24x24 像素的机场 RGB 点阵显示单元, 大概是这么个东西:

![Front](/assets/display-24x24-front.jpeg)

这种显示单元在机场的航站楼里面经常见到，用来显示航班信息, 一个单元显示一个汉字. 常见是红色或黄色单色字符, 特点是显示效果非常醒目.

![Airport Screen](/assets/display-24x24-airport-screen.jpeg)

听商家说, 这一批屏幕来自于浦东机场. 大概是机场航班信息显示屏幕的一部分. 如上图, 一个汉字正好使用一个显示单元.
字体边缘清晰, 亮度高.

简单搜索发现相关似乎 IC 资料能搜到, 猜测驱动应该不难, 所以当时就收了下来. 垃圾佬的特色就是赌.

## 介绍

整个显示单元成色还可以, 但设备常年在较恶劣使用环境下工作, 有不少陈年灰, 需要清理.

正面稍有岁月痕迹, 外壳有些许运输划痕, 但屏幕成色很好. 屏幕隐约能看到大块的像素点.

![Back](/assets/display-24x24-back.jpeg)

背后是巨大的散热金属片, 用于给 RGB 背光驱动板散热. 说明 RGB 背光板功耗不小, 提醒我们不要直接用单片机驱动.
从背面的文字可以确定屏幕型号是 KD54008-L025. 同时插座文字可以确定屏幕的上下方向.
左右两侧有三个 PH2.0 接口, 用于背光级联驱动和背光电源. PCB 上有标注, 但被散热片遮挡不容易看到.
背光板上有三个电位器可以调整 R, G, B 的亮度.

![Back Light Driver PCB](/assets/display-24x24-back-light-board.jpeg)

拆掉后可以看到背光 RGB 灯珠, 分十六组, 使用移位寄存器驱动, 即完全改变一片背光板的颜色, 需要通过移位16次.

![LCD Screen Teardown](/assets/display-24x24-screen.jpeg)

屏幕是两层静态 TN, 每一层都是 24x24 像素, 相同. 这种屏幕的特点是响应速度快, 高对比度, 叠两层对比度更高.
LCD 驱动板分上下两块 PCB 板, 每块板四个 LCD 驱动芯片 LC7931 级联. TSSOP20 封装的是 74HC245, 用于级联信号驱动能力.
上下两块驱动板的布局近似, 但移位方向和屏幕像素连接稍有不同, 基本上是镜像对称的. 这里放上板的图:

![LCD Driver PCB](/assets/display-24x24-lcd-driver-board.jpeg)

每块驱动板背后有两个 10pin 接口, 用于级联和电源. 10pin 接口是超薄接插连接器, 不是常见的型号, 但可以用 MX1.25-10P 超薄接头兼容.

折腾这种东西像是在复习自己当年的数字电路知识, 高低电平, 时钟, 包括移位寄存器, 锁存器等等.

## 电路部分

掏出家家必备的万用表开始, 一顿测.
背光驱动板因为接口有标注, 可以直接给驱动信号. LCD 驱动板稍微复杂一点, 但你只需要一个周末的下午. 有一些常用的经验:

- 如果外壳或安装孔接地, 可以很容易确定地线和电源
- 从已知芯片和资料比较完善的芯片开始, 检测信号引脚, 例如 74HC245
- 级联式组合的芯片走线一般都比较有规律, 可以大致猜测

背光和LCD驱动是两部分独立的电路,分别处理。

### LCD 驱动板

单个单元有上下两块驱动板, 分别用于上下两半屏幕. 两块驱动板的布局近似, 但移位方向和屏幕连接稍有不同, 基本上是镜像对称的.

以上板为例(01A):

- 4 片级联 LC7931, 用于驱动屏幕, 三洋的 80-channel Liquid-crystal Display Driver
- 1 片 HC245A , 即熟悉的 74HC245, Octal 3-State Noninverting Bus Transceiver, 三态 8 位总线收发, 用于驱动信号和级联信号的 buffer, 增加级联信号驱动能力
- 两个超薄 10pin 板对线接插连接器, 不是常见的型号, 能近似兼容的信号是 MX1.25-10P 超薄接头(必须是超薄)

通过分析74HC245三态8路总线收发芯片的使用情况,可以快速判断该PCB的供电和信号线路。74HC245的使能信号和方向信号均接地,说明它被当作一个普通的缓冲芯片使用,用于增加级联信号驱动能力.

74HC245的8路信号输入输出分别引向其他部分。由于LTC7931支持级联, 需要4条信号线控制, 因此可合理猜测这8路信号是用于TC7931的级联输入输出信号.

LTC7931是 80 通道液晶显示驱动芯片,用于驱动LCD的的像素点。引脚较多, 需要慢慢对应到接插件端口上.
LC7931 手册很详细, 是影印版的 PDF. 我们常把这种显示单元称为 "段码屏", 屏幕被分成了很多 "segment", 具体组织方式可以是规则矩阵排列, 也可以是类似数码管, 比如常见的七段数码管就是一种 segment display.

忽略74HC245, 简化电路示意图如下:

![SCH](/assets/display-24x24-lcd-driver-sch.jpeg)

主要是级联和接插件的信号输入输出。上下两块驱动板的四个接口的GND均位于显示单元外侧,电源输入位于靠模块中间一侧。

具体的像素布局需要在代码中一位一位修改尝试, 这里我结合了代码和万用表, 屏幕布局大概是:

- 屏幕上半部分像素 width=24, height=12, 由 4 片 LC7931 驱动, 级联方式
- 每片 LC7931 驱动 72 个像素点(segment), 80-bit 输出的高八位为 NC
- 72 个像素点正好是 6 列, z 字形排列, 12 行
- 屏幕下半部分与此镜像对称, 所以成了 80-bit 输出的低八位 NC
- 上下两块驱动板的布局通过跳线电阻选择移位方向, 所以大概是:

```text
(Back View)
+--CN0------------CN1--+
| A1 -> A2 -> A3 -> A4 |
|                      |
........................
|                      |
| B1 -> B2 -> B3 -> B4 |
+--CN0------------CN1--+
```

其中每个 Ax, Bx 表示单片 LTC7931 驱动的 72 个像素点, 也就是 6 列 12 行的像素点.

CLKSR 是移位寄存器时钟, 最高 1MHz, 计算可以得到, 80 segment 驱动, 一个屏幕 8 片 LC7931,
(1 MHz / (80 * 8) = 1562 Hz) 理论最高一个单元屏幕刷新率, 3 个级联的话就是 320 Hz. 速度还可以.
对于机场静态文字来说, 这个速度是足够的.

### 背光电路

背光板上的 16 组 RGB LED, 每组若干高亮三色 LED, 通过移位的方式逐渐驱动, 一个单元有三个连接器. 很常见的 PH2.0.

- CN1: 电源输入, 5V, GND, 实测单个单元最高电流可达 1A 以上, 常见单片机板子 5V 直接驱动可能过流烧坏电路!!
- CN2: 驱动信号输出, 用于级联
- CN3: 驱动信号输入
- 3 个电位器旋钮, 分别用来控制 R, G, B 的亮度, 方便调整颜色一致性

CN2, CN3 都是 8pin PH2.0 接插连接器. CN3 输入驱动信号如下:

| 自上到下 | 信号 | 说明 |
| --- | --- | --- |
| 1 | CLK | 背光移位寄存器时钟 |
| 2 | LATCH/LOAD | 背光移位寄存器锁存 |
| 3 | EN | 背光输出使能, 低电平有效 |
| 4 | R | 红色 |
| 5 | G | 绿色 |
| 6 | B | 蓝色 |
| 7 | GND | 地 |
| 8 | NC |  |

### 电路引出

得知了接口的信息, 就可以准备转接板了. 我这里选用了常见的单面洞洞焊接板, 引出电源及 10 pin 接口到 2.54mm 排针.

## Coding Time

MCU 这里选择 RPi Pico(RP2040), 开发框架选择 Rust Embassy. 它为嵌入式环境提供了方便简洁的 async/await 支持.
官方对 STM32, RP2040, NRF 等常见芯片提供了支持. 我目前正在做的 [ch32-hal] 项目提供了 WCH 32 位单片机的 Embassy 支持,
[hpm-hal] 项目提供了 HPMicro 的 32 位单片机的 Embassy 支持.

我们以 Embassy async task 的方式编写屏幕相关驱动.

从官方项目仓库直接找到 examples 目录, 复制出来搞定项目模板, 开始写代码.

### 背光驱动

先从简单的开始, 背光的移位和锁存可以快速验证, 甚至本例中, 我是用跳线来回短接模拟时钟测试的.

### LCD 像素驱动

相对来说 LCD 的驱动需要处理像素位置等信息, 额外还有 M 信号用于刷新 LCD, 相对复杂一些.

LCD M 信号的 AC 刷新, 可以使用 PWM 输出, 也可以偷懒直接用 GPIO:

```rust
#[embassy_executor::task]
async fn lcd_ac_driver(pin: AnyPin) {
    let mut pin = Output::new(pin, Level::Low);
    let mut ticker = Ticker::every(Duration::from_millis(10));
    loop {
        ticker.next().await;
        pin.toggle();
    }
}
```

移位式像素屏幕的显示内容驱动其实有完整的一个套路, 例如 WS2812 矩阵, 还有这种 LCD segment 驱动.
用 Framebuffer 是最简单的方式. Framebuffer 中字节的内容建议在空间允许的情况下, 尽可能接近最终传输输出的数据,
而不要为了方便像素写入逻辑使用复杂的发送时像素映射运算.(高速移位时候避免复杂指令对于多数低端单片机来说是必要的)

```rust
// 3 display units
// 10 byte per chip, 24 chip, 12 upper chip, 12 lower chip
pub struct Pixel24x24 {
    buf: [u8; 10 * 8 * 3],
}

impl Pixel24x24 {
    pub fn new() -> Self {
        Self { buf: [0; 10 * 8 * 3] }
    }

    pub fn set_pixel(&mut self, x: u16, y: u16, on: bool) {
        let x_index = x / 6;
        let y_index = y / 12;

        // zig-zag shape
        let chip_index = if y_index == 0 { (12 + x_index) } else { (x_index) };

        let start_index = 10 * (chip_index as usize);

        let mut chip_buf = if y_index == 0 {
            &mut self.buf[start_index..start_index + 9]
        } else {
            &mut self.buf[start_index + 1..start_index + 10]
        };

        let chip_x = x % 6;
        let chip_y = y % 12;

        let chip_n = chip_y + chip_x * 12;
        let byte_index = (chip_n / 8) as usize;
        let bit_index = chip_n % 8;

        if on {
            chip_buf[byte_index] |= 1 << bit_index;
        } else {
            chip_buf[byte_index] &= !(1 << bit_index);
        }
    }
}
```

随后实现 [embedded-graphics] 的 `DrawTarget` trait:

```rust
impl OriginDimensions for Pixel24x24 {
    fn size(&self) -> Size {
        Size::new(24 * 3, 24)
    }
}
impl DrawTarget for Pixel24x24 {
    type Color = BinaryColor;

    type Error = core::convert::Infallible;

    fn draw_iter<I>(&mut self, pixels: I) -> Result<(), Self::Error>
    where
        I: IntoIterator<Item = Pixel<Self::Color>>,
    {
        for Pixel(coord, color) in pixels {
            if self.bounding_box().contains(coord) {
                self.set_pixel(coord.x as u16, coord.y as u16, color.is_on());
            }
        }
        Ok(())
    }

    fn clear(&mut self, color: Self::Color) -> Result<(), Self::Error> {
        if color.is_on() {
            self.buf.fill(0xff);
        } else {
            self.buf.fill(0);
        }
        Ok(())
    }
}
```

完成 Framebuffer 之后, 驱动只需要把 FB 的内容移位输出即可:

```rust
#[inline]
fn shift_out_lsbf(p: &mut Output, clk: &mut Output, data: u8) {
    for i in 0..8 {
        if data & (1 << i) != 0 {
            p.set_high();
        } else {
            p.set_low();
        }
        Delay.delay_us(100);
        clk.set_low(); // falling edge shift data out
        Delay.delay_us(100);
        clk.set_high();
    }
}
```

由此, 就可以用 [embedded-graphics] 的 `DrawTarget` trait 来驱动 LCD 了.

![Char Display](/assets/display-24x24-char-display.jpeg)

## 进阶 - Field Sequential RGB Driving

在群里经 [wenting] 大佬的点拨,
发现它完全可以用 Field Sequential RGB Driving, 也就是分时复用的 RGB 驱动方式, 通过快速切换 R, G, B 三种颜色的亮度来合成出各种颜色.
它与传统的像素并行驱动显示技术不同，采用时间分割方法来处理颜色。

首先显示所有像素的红色成分，然后是绿色，最后是蓝色。这些颜色的显示通常以非常高的频率交替进行，以便人眼无法察觉到颜色的切换，而是感知到这些颜色的混合效果。由于人眼具有持续性的视觉特性（视觉暂留），不同颜色的快速切换可以在观众的视觉中自然地混合起来，从而形成完整的图像。

本文配图即为这种分时复用 RGB 驱动方式的效果.

## 后

其实这个拆机折腾早在 2023 年末即完成, 只不过迟迟一直没有总结.

后来考虑到 5V 输出更稳定, 单独做了一块 CH32X033(X035) 的板子, 方便焊接排线驱动, 灵感来自 DALL-E 的一次输出, 即结合可焊接 pad 和 2.54mm 排针, 方便使用.

![PCB Board Design](/assets/display-24x24-pcb-board.jpeg)

[hpm-hal]: https://github.com/hpmicro-rs/hpm-hal
[ch32-hal]: https://github.com/ch32-rs/ch32-hal
[wenting]: https://www.youtube.com/watch?v=t_YXjM7Keqw
[embedded-graphics]: https://docs.rs/[embedded-graphics]/latest/embedded_graphics/
