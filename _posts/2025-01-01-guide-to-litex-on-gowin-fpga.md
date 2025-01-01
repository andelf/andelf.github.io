---
layout: post
title: "LiteX RISC-V 软核入门指南 - 使用 Gowin FPGA // Getting Started with LiteX: A Beginner's Guide to Soft RISC-V IP Core Workflow Using Gowin FPGA"
date: 2025-01-01 10:38 +0800
tags:
  - embedded
  - fpga
  - risc-v
  - rust
toc: true
published: false
---

> 难度: 入门. 本文介绍 LiteX 的基础性内容, 最终实现一个在 FPGA 上运行的 RISC-V 软核, 并基于此编写简单嵌入式代码.
> 读者应具有 FPGA 和嵌入式编程的基础知识. 对 Verilog 或其他 HDL 语言有一定的了解.

前段时间在立创商城购买了一块 [立创开发板] 出品的 Gowin GW2A-18C FPGA 开发板, 名字很喜人, 逻辑派 [LCKFB LJPI].
这是立创开发板系列出的第一块 FPGA 开发板, 使用了 GD32 MCU + FPGA 的组合, 两者之间提供了 GD32 的 PA0~PA7 的连接,
支持八位总线, 或 QSPI0, USART1 实现两者的通信.

简明参数如下:

- GW2A-LV18 PG256C8/I7
  - 逻辑单元(LUT4): 20,736
  - 静态随机存储器ESRAM(bits): 828K
  - SSRAM(bits): 40K
  - 寄存器(FF): 15,552
  - 乘法器(18x18 Multiplier): 48个
  - 锁相环PLLs: 4
  - I/O数量: 207
- GD32F303CBT6
  - 内核: Cortex-M4
  - 工作频率: 120MHz
  - FLASH: 128KB
  - SRAM: 32KB
- DDR3 SDRAM: MT41J128M16JT
  - 2Gbit, 256MB
- SPI Flash: W25Q64
  - 64Mbit, 8MB
- Peripherals
  - USB via FPGA
  - HDMI out
  - 8 位数码管, 共阳
  - RGB LED, 1 for MCU, 2 for FPGA
  - 按键若干
  - TFT LCD FPC socket for MCU
  - SD Card for MCU SPI2

随板子赠送了一个 CH32V 的 JTAG/SWD 双模式调试器, 下载器固件不开源, 实现方式是软件模拟 FT2232. 有点类似 Sipeed 家的 Gowin FPGA 开发板, 他们使用 BL702 或者 BL616 来模拟 FT2232.
测试下来, 该下载器稳定性很好, 驱动完美.

总的来说, 这块逻辑派开发板外设丰富, MCU+FPGA 的组合很有想象力, 但也有若干设计不足, 个人观点如下:

- FPGA IO BANK 供电电压锁死 3V3, 不可调
- SD Card 接口给了 MCU, 这个设计有点失败
- 供电设计有所不足, 例如排针引出的 3V3 只允许输出禁止输入, 板子对多路供电的支持可能有所欠缺
- MCU 和 FPGA 之间的接口略少, 无法实现 8 位数据总线 + WR/RD/CK 式总线接口(可以通过排针处 IO 短接)
- MCU USB 接口未引出, 只能通过扩展底板实现

虽然有以上缺点或不足, 但这不妨碍它成为一块可玩度极高的综合性开发板.

本文不会对 MCU 部分编程做过多涉及.

## 介绍

[LiteX] 是一个开源的嵌入式 FPGA 框架, 为 FPGA 下的软核开发提供了开箱即用的支持. 具体说来他包括:

- 诸多开箱即用的开发板预定义文件, 方便快速组合成一个 "SoC"
- 多个开源或闭源 CPU 软核定义, 及总线胶水代码
- 各种 FPGA 的编译和烧录工具链支持, 包括开源工具链 yosos + nextpnr + openFPGALoader
- 各种 FPGA 内置 IP Core 的胶水代码, 例如 PLL, xxx PHY 等
- 内置各种常见外设 IP Core, 例如 uart, gpio, dram, spi, sdcard, eth, video, usb, pcie 等等
- 一个功能丰富的软核 BIOS (boot loader), 支持串口交互和串口下载代码并执行 `litex_term`
- 其他组件: 模拟器, 调试器,

就目前所知, 它的支持是最全的, 也是入门相对最简单的 FPGA + Soft IP Core 一站式解决方案.

LiteX 内部组件多使用 [migen] 编写, 这是一个 Python 方言的 HDL, 或者也有称之为 HDL code generator.
因为它最终编译为 Verilog. 传统的 HDL 如 Verilog, VHDL 编写较为繁琐, 有诸多误区, 新手需要一定的经验才能避开.

通过使用 HDL code generator, 可以使用更强的检查和设计规范来避免出错,

其他 HDL code generator 还有大家熟知的 Chisel, SpinalHDL, Bluespec 等等.

[Gowin] 提供了完整的工具链, 包括 IDE 和下载器程序. 支持 Windows / Linux / macOS 平台. (其中 macOS 目前只提供教育版本, 且需要修复链接错误).
一般来说, 厂商提供的工具链除了 IDE 图形化访问之外, 还会有一个命令行接口, 甚至多数时候, 图形化界面只是对应的命令行工具的二次封装,
例如在 [Gowin] 工具链中, `gw_sh` 就是底层的命令行工具, `programmer_cli` 是命令行版的编程烧录工具.
`gw_sh` 是一个基于 Tcl 的命令行工具, 支持配置 Gowin FPGA 项目, 对齐进行综合布线, 生成最终的 Bitstream 文件和所有中间结果.

[Yosys] 是开源 FPGA 工具链, 对 Gowin 的 FPGA 芯片有着不错的支持. 完整工具链可以从 [oss-cad-suite-build](https://github.com/YosysHQ/oss-cad-suite-build)
下载. [nextpnr](https://github.com/YosysHQ/nextpnr) 即 "place & route". 使用 yosys + nextpnr 工具链可以实现纯开源环境的 FPGA 开发.

当然, 开源工具链在功能支持和优化上有诸多不足, 实际开发学习中, 建议还是使用官方 [Gowin] 工具链, 减少因为工具 bug 导致失败的可能.

[openFPGALoader] 顾名思义, 是一个开源的 FPGA 烧录工具, 支持众多 FPGA 厂商, 不过在本文编写时候, openFPGALoader 在 macOS 下似乎不能支持逻辑派的烧录.

## Refs

[立创开发板]: https://lckfb.com/
[LCKFB LJPI]: http://wiki.lckfb.com/zh-hans/fpga-ljpi
[LiteX]: https://github.com/enjoy-digital/litex
[Gowin]: https://www.gowinsemi.com.cn/
[openFPGALoader]: https://github.com/trabucayre/openFPGALoader
[Yosys]: https://yosyshq.net/yosys/
