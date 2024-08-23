---
layout: post
title: 'Rust 嵌入式开发中的外设寄存器访问：从 svd2rust 到 chiptool 和 metapac - 以 hpm-data 为例'
date: 2024-08-23 09:51 +0800
author: andelf
tags:
  - embedded
  - rust
  - embassy
toc: true
published: true
---

Embedded Rust Peripheral Register Access: svd2rust, chiptool and metapac Approach.

本文是基础向文章, 介绍了 Rust 嵌入式开发中的外设寄存器访问问题，以及社区提供的解决方案。包括以下内容:

- 简短历史回顾
- [svd2rust] + [svdtools] 工作流
- 介绍 [Embassy] 框架中 stm32-metapac 所使用的 [chiptool]
- metapac 的设计与实现 - 以 [hpm-data] 和 [hpm-metadata] 为例
- 额外内容: pac 库的其他内容

## 背景

在嵌入式开发中，我们经常需要访问系统外设寄存器，以配置外设的工作模式、读取传感器数据等。在 C 中, 我们通常使用宏定义和来访问外设寄存器，例如：

```c
uint32_t temp = ptr->ADC16_CONFIG0;
temp |= ADC16_ADC16_CONFIG0_REG_EN_MASK
         |  ADC16_ADC16_CONFIG0_BANDGAP_EN_MASK
         |  ADC16_ADC16_CONFIG0_CAL_AVG_CFG_MASK
         |  ADC16_ADC16_CONFIG0_CONV_PARAM_SET(param32)
ptr->ADC16_CONFIG0 = temp;
```

其中 `ptr` 类型为 `ADC_Type *`，`ADC_Type` 是一个结构体，包含了 ADC 模块的所有寄存器字段, 按照相应内存布局一一映射。字段往往定义为 `volatile` 类型，以确保编译器不会对其进行优化。

更原始的, 比如在 8051 等单片机上, 往往直接通过内存地址来访问外设寄存器或 SFR, 例如:

```c
#define ADC16_CONFIG0 (*(volatile uint32_t *)0x4000_0000)
uint32_t temp = ADC16_CONFIG0;
temp |= ADC16_ADC16_CONFIG0_REG_EN_MASK
         |  ADC16_ADC16_CONFIG0_BANDGAP_EN_MASK
         |  ADC16_ADC16_CONFIG0_CAL_AVG_CFG_MASK
         |  ADC16_ADC16_CONFIG0_CONV_PARAM_SET(param32)
ADC16_CONFIG0 = temp;
```

虽然这种方式简单直接，但是不够安全，容易出现错误。例如，当字段名误用时，编译器往往不会报错，而是直接生成错误的代码。另外，当字段的位宽和偏移写错时，也会导致错误的配置.
对于嵌入式环境来说, 更难以调试. 究其原因, 一是因为 C 语言中的宏是朴素的文本替换, 缺乏类型检查, 二是因为 C 语言中的类型系统较弱, 隐式类型转换较多. 另外还有历史原因, C 语言中的指针操作较为灵活, 这种 `struct` + 宏定义的方式在各大芯片厂商的 SDK/HAL/LL 中被广泛使用.

在 Rust 中，我们同样可以通过类似的直接操作内存地址映射的方式访问外设寄存器。这种方式的优点是速度快，但缺点是不够安全，容易出现错误。为了解决这个问题，社区提供了 [svd2rust] 或 [chiptool] 等工具工具来生成类型安全的外设寄存器访问代码.

## 由来

这里会绍一个虚拟的发展历程, 可能并不代表真实的历史发展过程, 也不代表新方案完全替换了旧方案.

### 源起 - unsafe & volatile memory access

在 Rust 中，我们可以通过 `unsafe` 代码块和 `ptr::read_volatile`、`ptr::write_volatile` 等函数来访问外设寄存器。例如：

```rust
let ptr = 0x4000_0000 as *mut u32;
unsafe {
    let temp = ptr::read_volatile(ptr);
    ptr::write_volatile(ptr, temp | 0x1234);
}
```

这种方式的优点是简单直接，但缺点是不够安全，且需要依靠开发者本身的经验和代码命名规范来确保字段, SET, MASK 等的正确性. 直觉上, 就是在 Rust 中写 C 的 feel.

### Memory Mapped Register IO - MMIO

和上文提到的 C 结构体类似, Rust 中也可以定义类似的结构体来映射外设寄存器。例如：

```rust
#[repr(C)]
pub struct ADC {
    pub config0: Config0,
    pub config1: Config1,
    pub data: u32,
    // ...
}

let adc = 0x4000_0000 as *mut ADC;
unsafe {
    let rb = unsafe { &mut *adc };
    // calling method or write to `rb.config0`
}
```

和 C 不同的是, Rust 缺乏 bitfield 的语法糖, 也就是说, Rust 中的结构体字段访问无法直接精确到 bit, 至少也是 `u8`. 但这并不妨碍 Rust 社区创建各种好用的第三方 crate, 例如
[bitfield], [bit_field] 等. 通过直接使用 bitfield 作为字段类型, 可以更加直观的访问寄存器字段. 同时还有 [bitflags] 等 crate 提供类似 C 中标志位操作的功能.

这种方式的安全性有所保证，也一定程度上支持 C 样式的代码直接翻译. 但缺点是需要手动定义结构体和字段类型, 工程量大, 且容易出错.

另外在实际使用中, 还需要处理 `volatile` 的问题. 避免编译器优化掉对寄存器的访问.

## svd2rust

[svd2rust] 是一个由 Rust 社区提供的工具，用于将 SVD 文件转换为 Rust 代码。SVD 文件是一种 XML 格式的文件，用于描述芯片的外设寄存器。svd2rust 会根据 SVD 文件生成一个 Rust 模块(`xxx-pac`)，包含了芯片的所有外设寄存器的访问代码. 具体来说就是

- 每个外设映射为一个 `periph::RegisterBlock` 结构体提供寄存器访问
- 每个寄存器字定义为一个 RegisterBlock 的字段(或成员函数), 通过 `read`, `write`, `modify` 方法来访问: "read proxy" and "write proxy"
- 单个寄存器被定义为类似 bitfield 的结构体
- 寄存器位的访问被分为 `read`, `write`, `modify` 方法, 其中 `write`, `modify` 通过闭包来传递具体的操作

### svd2rust 寄存器访问示例

早期 svd2rust 实现直接使用了 MMIO struct 的方式, 生成的代码例如:

```rust
#[doc = r"ADC Register block"]
#[repr(C)]
pub struct RegisterBlock {
    #[doc = "0x00 - status register"]
    pub stat: STAT,
    #[doc = "0x04 - control register 0"]
    pub ctl0: CTL0,
    #[doc = "0x08 - control register 1"]
    pub ctl1: CTL1,
    // ...
}
```

访问时使用:

```rust
let rb = unsafe { &mut *pac::ADC0::PTR }; // `pac` 是生成的模块

let val = rb.stat.read().adc_stat().bits();
rb.ctl0.write(|w| w.adc_en().set_bit().adc_start().set_bit());
rb.ctl0.modify(|_r, w| w.adc_en().clear_bit());
```

后来 svd2rust 在一次更新后, 将所有字段的 `pub` 属性去掉, 转而使用 const fn 来访问寄存器字段. 例如:

```rust
///Register block
#[repr(C)]
pub struct RegisterBlock {
    statr: STATR,
    ctlr1: CTLR1,
    ctlr2: CTLR2,
    // ...
}
impl RegisterBlock {
    ///0x00 - status register
    #[inline(always)]
    pub const fn statr(&self) -> &STATR {
        &self.statr
    }
    ///0x04 - control register 1/TKEY_V_CTLR
    #[inline(always)]
    pub const fn ctlr1(&self) -> &CTLR1 {
        &self.ctlr1
    }
}
```

访问时使用:

```rust
let rb = unsafe { &mut *pac::ADC0::ptr() }; // `pac` 是生成的模块

let val = rb.statr.read().adc_stat().bits();
let flag_val = rb.adc_stat().read().status().bit_is_set();
rb.ctlr1().write(|w| w.adc_en().set_bit().adc_start().set_bit());
rb.ctlr1().modify(|_r, w| w.adc_en().clear_bit());
```

这样做的好处是更好地隐藏了寄存器字段的具体实现，在嵌入式 Rust 中往往会大量使用 `unsafe`, 其中缺乏必要的检查, 通过隐藏字段的具体实现, 可以减少错误的发生.
同时, 通过 const fn 提供的字段访问, 可以很好地支持 "内存重叠字段", 例如在 USB 外设中, 不同模式下, 同一个寄存器地址的字段可能有不同的含义, 通过 const fn 可以很好地支持这种情况.

### 使用 svd2rust 生成 pac 库

svd2rust 工具可以通过简单的命令行调用来生成 pac 库. 直接 `cargo install` 即可安装.

但实际使用过程中, 往往有很多的额外工作, 例如:

**去哪里寻找 SVD 文件?**

大部分情况下 SVD 文件可以从芯片的 CMSIS pack, 芯片厂商的 SDK 中找到. 去芯片厂商的网站翻一翻也许能找到.
SVD 文件为 IDE 的调试功能提供了外设寄存器视图, 所以在对应的 IDE 或 IDE 扩展中也能找到.
同时, 直接联系厂商, 也许能得到帮助.

比如:

- 常见的 Cortex-M MCU 一般会提供 [CMSIS packs](https://www.keil.arm.com/packs/), 可以搜索芯片型号下载
- CH32/GD32 等国产芯片的 SVD 文件可以在 [MounRiver Studio](http://www.mounriver.com/download) IDE 的安装目录找到
- HPMicro MCU 的 SVD 文件, 在官方 [hpm_sdk](https://github.com/hpmicro/hpm_sdk/)

**SVD 文件的质量如何?**

SVD 文件往往由芯片厂商提供, 有些是由社区维护的, 质量参差不齐, 经常能见到格式报错, 字段错误等问题.
直接使用 svd2rust 工具执行转换也会提示报错信息, 当错误不够直观时候, 可以通过 `xmllint` 工具检查:

```shell
xmllint --schema svd/CMSIS-SVD.xsd --noout XX32XXX.svd
```

同时 [svdtools] 提供一套基于 YAML 格式的 SVD patch 工具, 可以用来修复 SVD 文件中的错误, 修改字段, 新增外设等等.

对于 [svdtools] 补丁工作流的使用, 可以参考 [stm32-rs], 或者规模较小的 [ch32-rs]. 基本思路是拿到官方 SVD ->
修正格式错误(这个没得洗, 毕竟 xml 库都读不进去的话没有办法处理) -> 创建 patch 文件 -> svdtools apply -> patch 后的 SVD 文件 -> svd2rust -> pac.

**SVD 文件的版权问题?**

SVD 文件往往是芯片厂商提供的, 有些芯片厂商会在 SVD 文件或对应下载包中加入版权信息, 有些则没有. 一般来说, 用于开发者开发软件, 一般不会有问题,
但考虑到 pac 库发布需要, 最好联系芯片厂商, 以确认是否可以使用, 以及是否可以把 SVD 源文件包含在 pac 库中.

**找不到 SVD 文件怎么办?**

如果厂商没有提供 SVD 文件, 也可以通过手动编写 SVD 文件, 但这需要对芯片的外设寄存器有一定的了解, 以及对 SVD 文件格式有一定的了解.
一般来说, 从芯片手册中可以找到寄存器的描述, 以及寄存器地址, 位宽等信息.

直接以 YAML 格式编写 SVD 文件, 也是一种选择, 请参考 [svdtools] 的文档.

## chiptool

[chiptool] 是一个由 [Embassy] 社区提供的工具，用于生成 Rust 外设寄存器访问代码, 主要用于 [stm32-data], 为 Embassy 框架提供 STM32 所有 MCU 的外设寄存器访问支持.
相关背景可以参考项目首页, 其中有详细的介绍. 要点如下:

- [chiptool] 其实是 [svd2rust] 的一个 fork, 使之更适用于创建 `metapac` 式的 pac 库, 即厂商的一系列不同芯片的所有外设寄存器都放在一个库中. 这样做的好处是可以更好地复用代码和元数据信息
- chiptool 没有使用 owned struct 的方式, 避免滥用 ownership, 提供更宽松的使用方式
- chiptool 没有使用字段的 read/write proy, 这样字段本身作为类型(`repr(u32)`)可以直接拿来保存寄存器值 - 一个常见场景是拿到中断 flags 值, 依次判断, 修改, 最后写回寄存器, 用来清除中断标志
- chiptool 没有使用 MMIO 结构体, 而是直接保存外设地址
- 提供了单个 YAML 文件表示一个外设的处理方式
- 提供更方便的 transform 支持, 用于合并寄存器块, 字段, enum 类型, 创建 cluster, array 等

### chiptool 寄存器访问示例

具体使用方法和 svd2rust 基本类似, bit field 访问方法略有不同, 通过 `set_xxx` 使用, 总体上更简洁:

```rust
let r = pac::ADC0;
let val = r.statr().read().0; // 读取寄存器值
let flag_val = r.adc_stat().read().status(); // 读取寄存器字段

r.ctlr1().write(|w| w.set_adc_en(true)); // 设置寄存器字段

r.ctlr1().modify(|w| w.set_adc_en(false)); // 修改寄存器字段, 闭包不再需要传递 `r` 参数, 读出的值直接通过 `w` 访问
```

### 使用 chiptool 生成 pac 库

相比之下, chiptool 更适合于生成 metapac 风格的 pac 库, 这也就意味着它的门槛更高, 需要更多的元数据信息, 以及更多的工作量.

曾经唯一的参考资料是 Embassy 项目维护的 [stm32-data]. 在它的基础上, 我裁剪并维护了 [ch32-data] 和 [hpm-data], 都可以作为 chiptool + metapac 工作流的参考.

管理多个, 乃至某一厂商所有 MCU 的外设寄存器访问代码, 需要对整个芯片系列有一定的了解, 以及对外设寄存器的共性和差异有一定的认识. 需要来回阅读参考手册和原始 SVD 文件, 以及对生成的代码进行测试.

而 svd2rust 目前需要额外的脚本或工具才能更好支持单个 pac 库对应多个芯片的情况, 例如 [form](https://github.com/djmcgill/form) 工具可以拆分 inline mode 到子 mod 文件.

## metapac 的设计与实现

这里将介绍 metapac 的创建步骤, 设计思路与具体实现细节, 方便读者理解 metapac 的流程, 并搞定自己的 metapac 库.
目前的规范一般是 `-data` repo 用于存放元数据和生成代码, `-metapac` crate 用于最终发布.

[stm32-data] 整个流程较复杂, 包含从多个数据源获取的元数据, 包括 SVD 文件, STM32CubeMX 数据文件, 官方 SDK 头文件, ST-MCU-FINDER 数据等,
然后从 SVD 提取外设寄存器描述 YAML 文件, 通过若干 crate 配合, 完成数据的整合, 生成 pac 库.

而一些其他厂家的 MCU 可能缺乏如此丰富的格式化元数据(json/xml/etc.), 可能需要手动维护.

所以针对这种情况 [ch32-data] 和 [hpm-data] 基于 [stm32-data] 的逻辑, 做了简化流程处理, 尽可能适合手工维护.
**例如针对不同 MCU family 的外设情况, 增加了 `include` 支持, 方便层级化管理外设**.

[hpm-data] 的难度相对更小一些, HPMicro 的 MCU 系列较少, 且外设跨度较小. 同时官方还提供了一些标准的元数据(官方 pinmux tool), 可以通过爬虫的方式拿到.

以下内容以 hpm-data 为例, 介绍 metapac 设计思路与具体实现细节.

### 项目目录结构介绍

- `d` 脚本, 封装各命令
- `data/` MCU Family, 外设寄存器元数据目录
- `hpm-data-serde/` MCU 元数据的 serde 结构定义, lib
- `hpm-data-macros/` proc-macro lib, 从结构体转 Rust 代码的依赖, 不需要定制, 从 stm32-data 复制即可
- `hpm-data-gen/` 所有元数据的解析和生成工具, 从 `data` 目录读取, 生成到 `build/data` 目录
- `hpm-metapac-gen/` 最终的 metapac 生成工具, 从 `build/data` 目录读取, 生成到 `build/hpm-metapac` 目录
  - `res/` 最后 metapac 的模板文件, 包括 `Cargo.toml`, `build.rs`, `METADATA` 常量结构体类型定义等等

`hpm-data-serde/` 并不是唯一的数据类型结构体定义, 它只用于保存到 `build/data` 目录的格式.
在 `hpm-metapac-gen/src` 下还有第二份, 用于从 `build/data` 下的 json 解析.
在 `hpm-metapac-gen/res/src` 下还有第三份, 用于在最终的 pac 代码中提供 METAPDATA 类型定义.

这是整个项目结构最绕的部分, 新手容易迷失在结构体定义报错中, 往往新增字段需要改三个地方.
但通过这种方式, 可以很好地分离数据定义, 数据解析, 数据生成, 以及最终的代码类型.
例如在最终的 METADATA 中, 很可能为了考虑嵌入式环境和常量类型的特点, 所有的字符串都会被转换为 `&'static str`, 所有的数组都会被转换为 `[u32; N]` 等等.

### 元数据准备

首先确定好需要做哪些目标芯片的 PAC, 如果范围较广, 需要提前预留扩展性(比如多核的情况, 不同子架构的情况).

#### Chip Family

不同的芯片系列列表数据可以从厂商网站获取, 也可以从多个芯片手册中获取. 创建 `data/chips/CHIP_NAME.yaml`. 芯片名称的具体的粒度可以根据芯片的外设共性来划分.
主要型号之后的额外后缀往往包含芯片的具体封装(package: QFN, BGA 等), 以及不同的温度等级, 电压等级, 批次等.
元数据的字段参考 `-data-serde` 内的定义即可. 我们把这个文件定义为单个芯片 pac 的所需全部数据入口:

这里以 `HPM5361.yaml` 为例:

```yaml
name: HPM5361
family: HPM5300 Series
sub_family: HPM5300, Single-core, Full Featured
packages:
  - name: HPM5361xCBx
    package: LQFP100
    pins: 100
  - name: HPM5361xEGx
    package: QFN48
    pins: 48
memory:
  - address: 0x00000000
    kind: ram
    name: ILM
    size: 128K
  - address: 0x00080000
    kind: ram
    name: DLM
    size: 128K
  - address: 0xf0400000
    kind: ram
    name: AHB_SRAM
    size: 32K
  - address: 0x80000000
    kind: flash
    name: XPI0
    size: 1M
cores:
  - name: RV32-IMAFDCPB # D25
    ip-core: Andes D25F
    peripherals: []
    interrupts: []
    include_peripherals:
      - "../family/COMMON.yaml"
      - "../family/HPM5300.yaml"
      - "../family/HPM5300_GPTMR23.yaml"
      - "../family/HPM5300_UART4567.yaml"
      - "../family/HPM5300_ADC1.yaml"
      - "../family/HPM5300_DAC.yaml"
      - "../family/HPM5300_OPAMP.yaml"
      - "../family/HPM5300_MCAN.yaml"
      - "../family/HPM5300_Motion.yaml"
      - "../family/HPM5300_PLB.yaml"
      - "../family/HPM5300_Secure.yaml"
    include_interrupts: "../interrupts/HPM5361.yaml"
    include_dmamux: "../dmamux/HPM5361.yaml"
    gen_dma_channels:
      HDMA: 32
```

除了芯片的基本信息, 还包括了芯片的内存布局, 而在 `cores:` key 下, 是外设列表, 中断列表, DMA 通道列表等等. 这里 cores 实现为一个列表, 以支持多核异构芯片, 虽然在 HPMicro 的 MCU 中并没有这种情况.

其中 `include_peripherals:`, `include_interrupts:`, `include_dmamux:` 是外设, 中断, DMAMUX 描述的直接引用, 这是对 stm32-data 的改进, 以支持更好的外设复用和手工维护.
`gen_dma_channels:` 表示 DMA 通道的数量, 用于生成 DMA 控制器和 channel 的元数据, 这些元数据可能会在 hal 实现中用到. 尤其是 [Embassy] 这种异步框架, DMA 通道的管理是一个重要的部分.

#### 外设元数据

外设的元数据是 pac 库的核心, 也是最复杂的部分. 一般来说, 一个外设的元数据包括:

- 外设的基本信息, 包括名称, 描述, 寄存器等
  - 寄存器块的定义, 包括寄存器地址, 寄存器名, 寄存器描述等
  - 寄存器字段的定义, 包括字段名, 字段位宽, 字段描述等
  - 寄存器字段的值定义, 包括字段值名, 字段值描述等, 通过 enum 提供
- 外设的中断信号
- 外设的引脚信号, 包括引脚复用情况
- 外设的时钟信号, 使能复位信号等
- 外设的 DMA 请求信息

其中寄存器的信息我们可以从 SVD 里获取, 通过 chiptool 提供的 `chiptool extract-peripheral` 子命令, 可以方便地从一系列 SVD 中生成对应外设的 YAML 文件.
之后的工作就是手工维护这些 YAML 文件. 对于不同芯片使用相同的外设, 可以通过文件 diff 的方法来判断是否同一外设.

其中 YAML 文件格式例子如下, 相比 SVD 的 XML 更简单明了, 便于维护:

```yaml
block/UART:
  description: UART0.
  items:
    - name: IIR2
      description: Interrupt Identification Register2.
      byte_offset: 12
      fieldset: IIR2
    - name: Cfg
      description: Configuration Register.
      byte_offset: 16
      fieldset: Cfg
    # .... other register items
fieldset/IIR2:
  description: Interrupt Identification Register2.
  fields:
    - name: INTRID
      description: Interrupt ID, see IIR2 for detail decoding.
      bit_offset: 0
      bit_size: 4
    - name: FIFOED
      description: FIFOs enabled These two bits are 1 when bit 0 of the FIFO Control Register (FIFOE) is set to 1.
      bit_offset: 6
      bit_size: 2
    # .... other fields
# ... other fieldsets
enum/RX_IDLE_COND:
  description: IDLE Detection Condition.
  bit_size: 1
  variants:
    - name: RXLINE_LOGIC_ONE
      description: Treat as idle if RX pin is logic one
      value: 0
    - name: STATE_MACHINE_IDLE
      description: Treat as idle if UART state machine state is idle
      value: 1
```

其中对寄存器描述的优化是整个工作最麻烦耗时的地方, 例如对 enum 的优化, 对寄存器 array 的优化等. 优化修改的好处是显而易见的, 例如如下两种代码风格对比:

```rust
// set PWM1_CMP7 mode
use hpm_metapac as pac;
use pac::pwm::vals;
pac::PWM1.cmpcfg(7).modify(|w| {
    w.set_cmpmode(0); // output compare
    w.set_cmpshdwupt(1); // on modify
});

// vs
pac::PWM1.cmpcfg(7).modify(|w| {
    w.set_cmpmode(vals::CmpMode::OUTPUT_COMPARE);
    w.set_cmpshdwupt(vals::ShadowUpdateTrigger::ON_MODIFY);
});
```

可见, 通过 enum 的方式, 可以更好地表达寄存器字段的含义, 免去额外注释, 也更容易理解, 更类型安全.

寄存器之外的其他信息一般需要从芯片手册中获取. hpm-data 在实现中, 大量使用了 [hpm_sdk] 头文件中的常量定义, 通过正则解析的方式动态填写在外设结构定义中.
另外前面提到的 pinmux tool 也提供了重要的引脚复用信息. 都通过 `-data-gen` 工具解析, 合并 Chip Family 信息后生成到 `build/data` 目录.

然后在 `family/` 中创建对应的外设版本引用就可以继而被 `CHIP_NAME.yaml` 的 include 引用.

```yaml
# part of family/HPM5300_UARTs.yaml
- name: UART0
  address: 0xF0040000
  registers:
    kind: uart
    version: v53
    block: UART
  # the following are filled by `-data-gen` tool
  # pins:
  # sysctl:
  # interrupts:
  # dma_channels:
```

#### `data-gen` 工具

`-data-gen` 工具会扫描 `data/chips/` 下所有 `CHIP_NAME.yaml` 文件, 处理 `include_x:` 引用, 根据对应的外设信息, 从 sdk 头文件中提取常量定义, 填充上述的 pins, sysctl, interrupts, dma_channels 等字段,
最终生成到 `build/data` 目录. 这个工具是整个 metapac 生成流程的核心, 也是最复杂的部分. 在 stm32-data 中, 也是这个工具从各种数据来源中提取生成结构化数据.

```shell
hpm-data-gen
├── Cargo.toml
└── src
    ├── main.rs # 主程序入口
    ├── dma.rs # DMA 通道信息提取
    ├── interrupts.rs # 中断信息提取
    ├── pinmux.rs # 引脚复用信息提取
    ├── iomux.rs # 引脚复用信息提取, 提出 sdk 常量
    ├── pins.rs # 引脚数量, GPIO port 等信息提取
    ├── registers.rs
    ├── sysctl.rs # 时钟, GROUP 使能信息提取
    └── trgmmux.rs # 处理全局 TRGMUX 信号
```

相关逻辑请参考 [hpm-data] 项目, 通过依次执行以上流程, 完成了外设元数据的准备. 最终元数据如下:

- `build/data/chips/CHIP_NAME.json` - 对应每个芯片的元数据
- `build/data/registers/periph_ver.json` - 对应每种外设的寄存器信息

### metapac 生成

`-metapac-gen` 工具会扫描 `build/data/` 下所有的芯片和外设寄存器数据, 生成最终的 metapac 代码. 结合 `res/` 下项目模板,
最终输出到 `build/hpm-metapac` 目录.
这个工具是整个 metapac 生成流程的最后一步, 直接输出一个 crate 目录作为结果.

- 处理外设寄存器信息, 生成 `periph_ver.rs` 文件
- 处理芯片信息, `METADATA` 结构, `metadata_xxxx.rs` 文件, 通过编译时 feature flag 选择引用具体文件
- 输出 `-metapac` crate

芯片信息的处理主要包括 `METADATA` 的处理, 最终所有芯片名 feature gate 和外设版本的映射关系, 中断结构体, `memory.x`, `device.x` 等信息的生成.

`METADATA` 是区别于 metapac 和传统 svd2rust PAC 的一个重要特点, 用于提供芯片的元数据信息, 例如芯片的内存布局, 中断表, 外设及版本信息等.
通过 `"metadata"` feature gate 启用. 这个信息在 HAL 驱动中可能会用到, 例如 Embassy driver 需要动态创建 DMA 类型等.

外设寄存器信息的代码生成相关逻辑主要是调用 [chiptool] 完成, 通过一个简单的 IR (intermediate representation) 结构, 生成对应的 Rust 代码.

题外: chiptool 的这个从 YAML 定义生成 `.rs` 代码的逻辑其实非常有用, 使用场景不仅限于 pac 库, 例如一些公共 IP 外设的定义, 二进制协议的定义,
I2C 传感器寄存器格式的定义等等. 甚至诸如 RISC-V CSR 字段的定义都有可能使用到这种机制. 希望这种格式成为 Rust 嵌入式的某种标准. [yaml2pac] 就是这样一个尝试,
实现单个文件的代码输出. 社区也有类似的工作 [embassy-rs/chiptool#17](https://github.com/embassy-rs/chiptool/pull/17).

#### hpm-metapac 的扩展内容

HPMicro 提供了多个系列的高性能 RISV-V MCU, 包括丰富的外设资源和高速时钟, 其中较为复杂的是 SYSCTL 的资源管理, 引脚的 IOMUX, 以及 TRGMUX 外设互联等.
这些外设涉及到大量的 CLUSTER 寄存器(即外设的寄存器块通过二级, 三级索引的方式使用, 更好地组织资源), 而 [chiptool] 目前对这种  CLUSTER 索引支持并不完善,
无法解析具体索引名字, 例如 CPU0 时钟设置 `SYSCTL.CLOCK[CLK_TOP_CPU0].MUX`, 在 chiptool 只能识别为 `pac::SYSCTL.clock(0).read().mux()`,
丢失了其中最终要的 `<dimIndex>` 信息. 其在 SVD 的原始定义如下:

```xml
<dimIncrement>0x4</dimIncrement>
<dimIndex>cpu0,cpx0,rsv2,rsv3,.... </dimIndex>
```

相同情况的还有 IOC, GPIOM 等外设, 为了处理这种情况, hpm-data 对 pac 库做了扩展, 单独将必备的索引信息, 以 `pub const NAME: usize` 的方式提供.

这些 consts 通过 hpm-data-gen 解析, 最终由 hpm-metapac-gen 生成为对应 pac 的子 mod:

- `hpm_metapac::clocks::` 下的所有时钟，用于 SYSCTL.CLOCK
- `hpm_metapac::resources::` 下的所有 SYSCTL 资源
- `hpm_metapac::pins::` 下的所有 GPIO 及其 PAD，用于 IOC
- `hpm_metapac::iomux::` 下的所有 IOMUX 设置（FUNC_CTL）
- `hpm_metapac::trgmmux::` 下的所有 TRGM 常量定义

对于 PAC 库来说, 不仅仅是提供给 HAL 驱动使用, 而是同时能给最终用户一个方便安全的寄存器访问接口. 对于某些设计良好的外设, 寄存器访问更直接有效.
由此通过简单的方式就可以使用对应的外设(虽然丢失了部分类型安全, 不过加一个 enum 也很容易, 这里主要是等待上游 chiptool 实现该机制):

```rust
use hpm_metapac as pac;
use pac::{iomux, pins};

pac::IOC
    .pad(pins::PA25)
    .func_ctl()
    .modify(|w| w.set_alt_select(iomux::IOC_PA25_FUNC_CTL_PWM1_P_1));
```

### pac 库的其他内容

上述一节其实已经介绍了 PAC 库在标准的寄存器访问定义之外还有哪些内容, 这里再总结一遍:

- 中断静态结构体定义, enum 定义 - 用于在 `-rt` 库中使用, 链接到中断处理函数
- `device.x` 定义中断处理函数的链接符号, 和中断表结构体结合使用
- `Peripherals` owned struct, 用于通过 ownership 机制管理外设资源 - 仅 svd2rust
- `CorePeripherals` owned struct, 用于管理核心外设资源 - 仅 svd2rust + Cortex-M
- `memory.x` 定义内存布局 - 实际上由于应用各异, 可能不会被使用到, 通过 feature gate 启用
- 各种 METADATA 信息 - 仅本文提到的 metapac

## 总结及对比

- C 中的外设寄存器访问, 方式简单直接, 但容易出错, 不够类型安全, Rust 基于自己的类型系统, 可以提供更好的类型安全, 但需要额外的工具支持
- svd2rust 适合于单个芯片的 PAC 生成, 通过 `svd2rust` 命令行工具, 可以快速生成单个芯片的 PAC 库, 目前也是社区最常用的方式
- chiptool 适合于多个芯片的 PAC 生成, 通过 `chiptool` 和 `-data` 仓库, 可以为多个芯片生成 PAC 库, 适合于厂商的多个芯片系列的 PAC 生成. 同时具有 `METADATA` 的特性, 用于提供芯片的元数据信息

除此之外 [ral] / [ral-registers] 也提供了另一种基于宏的外设寄存器访问方案.

<!-- Refs -->

[svd2rust]: https://github.com/rust-embedded/svd2rust
[chiptool]: https://github.com/embassy-rs/chiptool
[Embassy]: https://embassy.dev/
[bitfield]: https://docs.rs/bitfield
[svdtools]: https://github.com/rust-embedded/svdtools
[hpm-metadata]: https://github.com/hpmicro-rs/hpm-metapac
[hpm-data]: https://github.com/andelf/hpm-data
[bit_field]: https://crates.io/crates/bit_field
[bitflags]: https://crates.io/crates/bitflags
[stm32-rs]: https://github.com/stm32-rs/stm32-rs
[ch32-rs]: https://github.com/ch32-rs/ch32-rs
[stm32-data]: https://github.com/embassy-rs/stm32-data
[hpm_sdk]: https://github.com/hpmicro/hpm_sdk
[yaml2pac]: https://github.com/embedded-drivers/yaml2pac
[ral-registers]: https://docs.rs/ral-registers/latest/ral_registers/
[ral]: https://docs.rs/ral/latest/ral/
