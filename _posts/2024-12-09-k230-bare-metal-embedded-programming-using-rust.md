---
layout: post
title: 基于 Rust 的 K230 裸机嵌入式编程 - K230 Bare-Metal Embedded Programming Using Rust
date: 2024-12-09 16:53 +0800
tags:
  - embedded
  - rust
  - risc-v
image: /assets/k230-boards.jpg
toc: true
published: true
---

> 难度: 中等 | 适用读者: 具备嵌入式系统基础知识

本文记录了在 K230 芯片上进行 Rust 裸机开发的过程. 从启动流程的分析, 固件格式的解析，到编写裸机 Rust 程序, 完善初始化代码, 再到实际的外设控制和功能实现，
和后续开发过程的优化方案, 都进行了探索.

本文相关代码库: [k230-bare-metal](https://github.com/andelf/k230-bare-metal)。建议参考早期提交记录如 [e15968040](https://github.com/andelf/k230-bare-metal/tree/e1596804045b95b2f639036e10653605f04c72a6) 进行学习。

## 项目背景 (Background)

之前从立创开发板获得了 [立创·庐山派K230-CanMV] 的测评机会, 另外我自己也有一块块 [CanMV-K230](https://wiki.youyeetoo.com/en/CanMV-K230) 开发板.

K230芯片是嘉楠科技推出的 AIoT SoC, 采用异构单元加速计算架构，集成了 2 个 RISC-V 算核心和 AI 子系统 KPU(Knowledge Process Unit).
按照时间线, 应该是市面上最早一批支持 RV 向量扩展 RVV 1.0 的芯片之一. 主要特点：

- 双核 RISC-V 处理器
  - Core 0: 64位 RISC-V (RV64GCB)，800MHz
  - Core 1: 64位 RISC-V，1.6GHz，支持 RVV 1.0 向量扩展
- 专用加速单元
  - KPU: AI 推理加速器，支持 INT8/INT16
  - DPU: 3D 结构光深度计算单元
  - VPU: 视频编解码器，支持 4K 分辨率
- 丰富的外设接口
  - 通信接口: UART×5、I2C×5、SPI×3
  - 存储接口: USB 2.0×2、SD/eMMC
  - 其他: GPIO×72、PWM×6、WDT/RTC/Timer

正常使用的话, 板子使用 [CanMV] 固件, 该固件兼容 [OpenMV], 是非常方便开发环境.
固件底层实现基于 [RT-Thread] Smart, CanMV 实现为 RT-Thread 的一个应用(MicroPython fork).
RT-Thread Smart(RT-Smart) 是支持用户态应用的 RT-Thread 版本, 适用于支持 MMU 的 SoC, 例如 K230.
另外早期版本的 CanMV 固件使用 Linux + RT-Thread + Micropython. 官方也有纯 Linux 版本固件.

本项目旨在探索:

1. MPU 与 MCU 在启动方式和使用模式上的区别
2. 如何使用 Rust 进行 MPU 芯片的裸机开发
3. K230 的底层启动机制和硬件特性

对于 MPU 及多数 MCU 来说都存在一个片上 Boot ROM 用于启动系统. 一般来说, Boot ROM 会初始化一些硬件(例如 SPI Flash, TF Card等), 将固件加载到内存
然后加载系统固件的第一行逻辑(例如 U-Boot). 之后用户提供的系统固件会初始化更多硬件及加载真正的操作系统.

所谓的裸机开发, 就是不使用操作系统, 直接在硬件上运行程序, 类似 MCU 直接在系统 Boot ROM 之后运行的方式.

## 启动代码分析 (Boot Code Analysis)

首先需要查看官方仓库 [CanMV] 的代码, 确定是否有不开源的部分. 尤其是核心的 U-Boot 和 RT-Thread/Linux 驱动部分.
另外对于 U-Boot 还需要确认第一阶段启动代码 SPL(Secondary Program Loader) 是否开源. 因为 SPL 往往用于初始化 DDR 等外设, 以及加载 U-Boot, 很多厂商不开源,
只提供二进制文件.

好消息是, 相关的代码都在 [CanMV] 仓库中, 且开源. 但是代码结构比较复杂, 需要一定时间阅读分析具体的启动流程和逻辑.

然而, 随着 ChatGPT 的出现, 我们可以更快地完成代码分析. 我曾自嘲, ChatGPT 早出现几年的话, 很多工具链都不需要存在.

这里只考虑 TF card 启动的情况, 系统固件在 TF 卡上, 片上 Boot ROM 加载固件到内存. 也就是我们的程序需要做到和 U-Boot 一样的事情, 包括 SPL 的功能.

> 注：TF 卡、SD 卡、eMMC 在协议层面基本相同，本文不做严格区分

### 上电复位到加载并执行用户固件

首先，Boot ROM 加载固件到内存。这一部分的逻辑是直接固化在芯片内部的 Boot ROM 中，属于无法控制的部分，因为 Boot ROM 的代码和逻辑都集成在芯片内部，无法被用户修改或干预。Boot ROM 内部的实现机制是通过读取 BOOT0 和 BOOT1 两个引脚的状态来判断启动方式。这两个引脚的电平状态决定了芯片在启动时从何种介质加载引导程序。

从芯片手册看, Boot ROM 的内存映射位置位于 0x9120_0000 ~ 0x9121_0000, 使用 SRAM 的前半部分 0x8020_0000 ~ 0x8030_0000. 这些信息可以通过裸机程序读取 sp/ra 等特征确认. 例如 Boot ROM 会将 sp 可用内存的最大值. Boot ROM 跳转到用户固件往往使用 call 指令, ra 会被设置为 Boot ROM 的返回地址.

Boot ROM 会按照预先设定的固定格式，从 TF 卡中加载固件（通常是 U-Boot）到内存中。具体来说，Boot ROM 会访问 TF 卡，按照特定的协议和数据格式读取固件文件，将其解码并存储到指定的内存位置 0x8030_0000.

当固件加载完成后，Boot ROM 将程序的执行权转移到刚刚加载到内存的固件上，也就是跳转执行 U-Boot。这标志着启动过程从 Boot ROM 阶段进入了固件（U-Boot）阶段。U-Boot 作为一个功能更为强大的引导加载程序，可以进一步初始化系统硬件、加载操作系统内核，以及执行其他用户定义的启动任务。

接下来是用户固件的启动(U-Boot).

K230 配备了两个 CPU，分别称为 CPU0（小核）和 CPU1（大核）。在启动过程中，当芯片的复位信号被解除后，嵌入式启动 ROM（BOOTROM）会在 CPU0（小核）上开始执行。这意味着 CPU0（小核）是第一个被激活的核心，它负责执行初始的引导程序，进行系统的基本初始化。与此同时，CPU1（大核）的解除复位（de-reset）过程是由 CPU0（小核）来控制的。也就是说，CPU0（小核）在完成自身初始化的同时，还需要发送指令来解除 CPU1（大核）的复位状态，使其开始运行。这种架构设计使得 CPU0（小核）不仅肩负着引导系统启动的重任，还掌控着 CPU1（大核）的启动流程，为双核协同工作奠定了基础。

U-boot 分为两阶段启动, SPL 和 U-Boot, SPL 用于初始化 DDR 等外设, 加载 U-Boot, U-Boot 之后的逻辑(OpenSBI, RT-Thread Smart), 我们此次不考虑.
从固件格式看, 这部分以固件分区的方式存在.

### 固件格式

为了让我们的固件被 Boot ROM 识别, 需要特定的固件格式. 各家的 SoC 此类方案不同, 有使用 fat32 固定文件名的, 有使用特定偏移的固定格式的.

K230 的 Boot ROM 会识别 TF 卡的固定偏移位置数据特征, 满足格式的固件会被加载到内存.
Boot ROM 已经初始化了 UART0, 会有简单的报错信息, 例如 "boot failed with exit code 19" 表示未找到 TF 卡, "boot failed with exit code 13" 表示固件格式错误等.

```text
00000000  +-------------+-------------+-------------+-------------+
          | ........... | ........... | ........... | ........... |  <- Partition table / any other data
          | ........... | ........... | ........... | ........... |
          +-------------+-------------+-------------+-------------+
00100000  | 4b 32 33 30 | 8c fc 02 00 | 00 00 00 00 | bf 8d 0f 38 |   <- Firmware header: |K230...........8|
          | MAGIC: K230 | Length      | Encryption  | SHA256 hash |   <- Encryption 0: non encryption, 1: SM4, 2: AES+RSA
          +-------------+-------------+-------------+-------------+
00100010  | 03 f3 87 07 | fa 1b d8 1d | 4f a0 cd a0 | 7b 54 35 bd |.  <- SHA256 hash cont.
          +-------------+-------------+-------------+-------------+
00100020  | 35 82 85 89 | 66 4d ac 27 | ca f8 56 49 | 00 00 00 00 |   <- SHA256 hash cont. + Padding
          +-------------+-------------+-------------+-------------+
00100030  | 00 00 00 00 | 00 00 00 00 | 00 00 00 00 | 00 00 00 00 |   <- Padding zeros
          +-------------+-------------+-------------+-------------+
          | ........... | ........... | ........... | ........... |   <- Padding zeros
          +-------------+-------------+-------------+-------------+
00100210  | 00 00 00 00 | 73 25 40 f1 | 2a 82 ae 84 | 93 01 00 00 |   <- Firmware data, length zero position
          | Version     | OpCodes     | Data        | Padding     |   <- Version: 0
          | ........... | ........... | ........... | ........... |   <- Firmware data, raw opcodes
          +-------------+-------------+-------------+-------------+
```

相关 C 结构定义位于 [CanMV] `src/uboot/uboot/board/kendryte/common/board_common.h`. 这里我们简化处理, 不加密固件, 版本号使用 0.

编写一个 Python 脚本完成固件 .img 的创建:

```python
#!/usr/bin/env python3
# genimage.py

import hashlib

MAGIC = b"K230"

def sha256(message):
    digest = hashlib.sha256(message).digest()
    return digest


VERSION = b"\x00\x00\x00\x00"

with open("./firmware.bin", "rb") as f:
    data = f.read()

intput_data = VERSION + data

data_len = len(intput_data)
raw_data_len = data_len.to_bytes(4, byteorder="little")

encryption_type = 0
encryption_type = encryption_type.to_bytes(4, byteorder="little")

hash_data = sha256(intput_data)

firmware = MAGIC + raw_data_len + encryption_type + hash_data

firmware += bytes(516 - 32)  # padding
firmware += intput_data

img = bytes(0x100000) + firmware  # image offset 0x100000

# fill 512 boundary, make sure the image size is multiple of 512
if len(img) % 512 != 0:
    img += bytes(512 - len(img) % 512)

with open("./firmware.img", "wb") as f:
    f.write(img)

print("len", len(img))
```

其中 `firmware.bin` 通过 objcopy -O binary 生成:

```sh
cargo objcopy --release -- -O binary firmware.bin && python3 genimage.py
```

固件的写入可以借助任何烧录工具, 包括 `dd` 命令.

## 开始编写裸机 Rust 程序 (Start Writing Some Bare-Metal Code)

搞定了固件的加载, SoC 的控制流程就可以交给我们的程序了. 这里我们使用 Rust 语言编写裸机程序.
从相关代码阅读得知, TF card 中代码被加载到了 0x80300000 ~ 0x80400000. 为了避免额外的不确定性, 可以直接使用 U-Boot 的 linker script.

由于缺乏芯片开发的第一手资料, 我们并不知道 Boot ROM 之后初始化的状态具体如何, 这时候只能靠猜测和尝试.

### 验证裸机执行 - UART

对于裸机编程来说, 需要初始化设备的初始状态, 包括堆栈 sp, 系统执行模式, 中断表, 中断开启等等. 这些工作通常由 `start.S` 或 `crt0.c` 完成.
极小初始化代码往往只需要设置堆栈 sp, 保证函数可以跳转调用执行. 在 sp 非法的情况下, 如果函数调用返回, 会导致异常.

由于没有 JTAG 调试环境(芯片支持, 我这里没有用 CK-LINK), 如何判断我们的代码是否被执行, 以及代码是否正确执行, 是一个问题.
这里我们可以使用 UART0 输出调试信息. 由于 Boot ROM 已经初始化了 UART0, 我们可以直接使用.

从 U-Boot 源码中的 dtsi 文件中可以得知, K230 大量使用了 DesignWare IP 的外设, 例如 UART0, SPI, I2C 等.
这些外设的具体寄存器手册可以从网上获得. UART 外设兼容 16550, 即我们熟悉的 PC 串口芯片.

可以直接使用 `global_asm!`, 串口打印字符来验证代码是否被执行. 例如:

```rust
#![no_std]
#![no_main]

global_asm!(r#"
.section .text.start
.global _start
     la sp, _stack_start
     call _start_rust
"#);

#[no_mangle]
pub extern "C" fn _start_rust() {
    loop {
        // UART0.THR = 'A'
        core::ptr::write_volatile(0x9140_0000 as *mut u32, 0x41);

        for _ in 0..100000000 {
            unsafe { asm!("nop") }
        }
    }
}
```

编译烧录如上代码, 不出意外, 你会在串口终端看到一串 A 字符. 这说明我们的代码被执行了.

### 访问外设寄存器 - PAC

Rust 嵌入式访问寄存器往往通过 PAC crate 的方式, 例如 `stm32xxxx-pac` crate. 但是由于 K230 是一个新的芯片, 没有相关的 PAC crate.
也不太可能有 SVD 文件供生成. 这里我选择了 [chiptool] 的方式, 使用 [yaml2pac] 工具完成 PAC crate 的生成. 手动维护外设寄存器的 YAML 定义.
关于 PAC 的访问, 请参考 [Rust 嵌入式开发中的外设寄存器访问：从 svd2rust 到 chiptool 和 metapac - 以 hpm-data 为例](https://andelf.github.io/2024/08/23/embedded-rust-peripheral-register-access-svdtools-chiptool-and-metapac-approach/) 一文.

相关 YAML 文件完全可以通过 LLM 协助从 PDF 手册 OCR.

使用 [yaml2pac] 工具可以方便地形成我们自己的 PAC 库:

```rust
#[path = "uart_dw.rs"]
pub mod uart;

pub const UART0: uart::Uart = unsafe { uart::Uart::from_ptr(0x9140_0000 as *mut ()) };
pub const UART1: uart::Uart = unsafe { uart::Uart::from_ptr(0x9140_1000 as *mut ()) };
pub const UART2: uart::Uart = unsafe { uart::Uart::from_ptr(0x9140_2000 as *mut ()) };
pub const UART3: uart::Uart = unsafe { uart::Uart::from_ptr(0x9140_3000 as *mut ()) };
```

通过简单的封装, 可以方便地访问外设. 相关的 PAC 维护在缺乏资料的情况下, 是比较困难的. 但是一旦完成, 可以大大提高开发效率.

### 方便调试 - println! 宏

有了外设寄存器定义, 此时可以写个完整的 HAL Driver, 也可以简单寄存器访问, 实现一个 `println!` 宏.

```rust
#[derive(Debug)]
pub struct Console;

impl core::fmt::Write for Console {
    fn write_str(&mut self, s: &str) -> core::fmt::Result {
        use pac::UART0;

        for c in s.as_bytes() {
            unsafe {
                while !UART0.lsr().read().thre() {
                    asm!("nop");
                }

                UART0.thr().write(|w| w.set_thr(*c));
            }
        }

        Ok(())
    }
}

#[macro_export]
macro_rules! println {
    ($($arg:tt)*) => {
        {
            use core::fmt::Write;
            writeln!(&mut $crate::Console, $($arg)*).unwrap();
        }
    };
    () => {
        {
            use core::fmt::Write;
            writeln!(&mut $crate::Console, "").unwrap();
        }
    };
}
```

有了 `println!` 宏, 我们可以方便地输出调试信息了! 大大提高了开发效率.

## 完善初始化代码 (Complete Initialization Code)

目前为止, 我们只初始化了堆栈, 系统的中断, 乃至 .bss 段都没有初始化. 这些在一个完整的嵌入式程序中都是必顼的.

和 MCU 编程不同的是, MPU 的代码执行是被 Boot ROM 加载到内存中某一区域的, 所以 MCU 中常见的 .data 区域 copy 是不需要的.
而 .bss 清零则依情况而定, 因为比较简单, 本节略过内存初始化部分.

### 中断处理函数

对于 RISC-V 来说, 中断处理函数是一个特殊的函数. Rust 提供了 `riscv-interrupt-m` ABI 专门用于中断处理函数的特殊逻辑.
具体就是增加了中断处理函数的栈帧保存和恢复, 以及使用 `mret` 指令返回.

```rust
#[link_section = ".trap"]
#[no_mangle]
unsafe extern "riscv-interrupt-m" fn _start_trap_rust() {
    println!("trap!");

    let mcause = riscv::register::mcause::read();
    println!("mstatus: {:016x}", riscv::register::mstatus::read().bits());
    println!("mcause:  {:016x}", riscv::register::mcause::read().bits());
    println!("mtval:   {:016x}", riscv::register::mtval::read());
    println!("mepc:    {:016x}", riscv::register::mepc::read());

    loop {}
}
```

这里打印了一些中断的重要信息, 协助判断中断函数是否被正常调用.

使用 `#[no_mangle]` 是为了暴露符号, 我们可以在汇编代码中使用这个符号设置中断处理入口地址.

使用 `#[link_section = ".trap"]` 是为了将这个函数放到 `.trap` 段, 以便于在链接脚本中处理, 尤其是内存对齐(`ALIGN(8)`).
这是写裸机代码常见错误, 因为 mtvec 寄存器的地址必须对齐(低2位由向量处理模式位占用), 否则会导致异常.

暂时我们不需要处理中断, 只需要观察中断是否被触发, 以及观察中断处理函数是否被执行. 所以使用 `loop {}`.

### 中断初始化

对于 RISC-V 来说, 中断的初始化大概需要如下步骤:

- 设置 mtvec: 中断处理入口地址
- 设置 mstatus 的 MIE 位: 允许中断
- 设置 mie 的 MEIE 位: 允许外部中断, 定时器中断等

K230 使用了 Xuantie C908 核心, 支持 CLINT 和 PLIC 中断控制器. 相关资料可以从 C908 手册中获得.

```rust
global_asm!("
    .option push
    .option norelax
    la gp, __global_pointer$
    .option pop

    la t1, __stack_start__
    addi sp, t1, -16

    la t0, _start_trap_rust
    csrw mtvec, t0

    call _early_init

    // 继续调用 _start_rust
    call _start_rust
");

#[no_mangle]
unsafe extern "C" fn _early_init() {
    {
        use riscv::register::*;

        mstatus::set_mie(); // enable global interrupt
        mstatus::set_sie(); // and supervisor interrupt
        mie::set_mext(); // and external interrupt
        mie::set_msoft(); // and software interrupt
        mie::set_mtimer(); // and timer interrupt
    }
}
```

mstatus 寄存器的 MIE 位用于控制中断使能, mie 寄存器的 MEXT 位用于控制外部中断使能, 例如 PLIC 中断.

这里同时初始化了 gp, 它是一个全局指针寄存器, 用于 Rust 的全局变量访问(在链接脚本中定义的特殊位置).
当然, 在使用内存区域较小较集中的时候, 很可能你不会见到使用 gp 寄存器的指令.

### 验证中断处理

我们可以通过直接触发软件中断的方式, 来验证中断处理函数是否被执行. K230 的 CLINT 中断控制器可以通过 `msip` 寄存器触发软件中断.

```rust
pac::CLINT.msip(0).write(|w| w.set_msip(true)); // trigger software interrupt
```

修改中断处理函数 `_start_trap_rust` 增加返回:

```rust
    if mcause.is_interrupt() && mcause.code() == riscv::interrupt::Interrupt::MachineSoft as _ {
        println!("Machine Software Interrupt");
        pac::CLINT.msip(0).write(|w| w.set_msip(false)); // clear software interrupt
        return;
    }
```

使用 mtime, mtimecmp 也可以验证定时器中断. 但我在使用中发现一个坑,
K230 的 CLINT 的 mtime 无法通过 64 位 load 指令读取, 读出内容随机. 不抛出任何异常.
这导致 64 位的 mtime 必须通过两次 32 位读取, 然后组合成 64 位.
只有 `rdtime` 指令可以一次读取 64 位 mtime.

### 其他 CSR 初始化

基于平台不同, 还需要初始化其他硬件, 例如关闭 PMP, 初始化 FPU, 开启 mcycle, mtime 计数器等等.
其中 FPU 的初始化是必须的, 否则任意浮点数指令会导致异常. Rust 的 `riscv-interrupt-m` 实现不够智能, 无法判断 FPU 的使用情况,
所以当 target 包含 `+f`/`+d` 时, ABI 会默认使用 FPU 压栈指令.

```rust
// 这里省略平台特定的寄存器初始化部分
// 包括关闭 PMP
asm!("
    li    t0, 0x00001800
    csrw  mstatus, t0");

mcounteren::set_cy(); // enable cycle counter
mcounteren::set_tm(); // and time counter

// FPU init
mstatus::set_fs(mstatus::FS::Clean);
mstatus::set_fs(mstatus::FS::Initial);
asm!("csrwi fcsr, 0");
```

`mstatus` 除了中断使能之外, 还负责当前 CPU 的运行模式, 例如 M/S/U 模式.

有了系统的 `mcycle`, 就可以方便地使用 `embedded-hal` 的 `Delay` trait, 实现较为精确的延时.

```rust
const CPU0_CORE_CLK: u32 = 800_000_000;

let mut delay = riscv::delay::McycleDelay::new(CPU0_CORE_CLK);
delay.delay_ms(1000);
```

### DDR init

DDR init / (SDRAM 初始化) 是一个比较复杂的过程, 一般需要初始化时钟, 复位控制器, PHY 训练, 芯片初始化, 时序配置, 自检等等.
这些内容往往都是厂商直接提供, 在相关的 DDR 初始化代码中, 相关寄存器写入流程也是如同天书一般.

所以 DDR init 代码直接通过 LLM 从 C 翻译. 不做额外解释. DDR 芯片不同, DDR init 代码也是不同的.

DDR init 之后, 我们就可以使用 DDR 区域的内存. 这里有个比较坑的地方, DDR 内存起始地址是 0x0000_0000,
然而 Rust 访问零地址有诸多限制, 多数函数会直接 panic. 程序中应该避免使用 0x0000_0000 地址.

## 正式开始裸机编程 (Start Real Bare-Metal Programming)

有了以上的初始化基础, 我们终于可以开始正式的裸机编程了. 例如初始化其他外设, 读写外设寄存器, 甚至是实现一些简单的功能.

这里以两个外设为例简单展示. 相关的外设寄存器定义我已经人肉写好在 [k230-bare-metal] 仓库中.

### GPIO 点灯

无论 MCU 还是 MPU, GPIO 点灯的步骤都是类似的:

- 使能(或复位) GPIO 外设时钟, 电源
- 设置引脚功能复用, 引脚模式
- GPIO write

K230 默认情况下外设的时钟和电源信号都是开启的(检查相关寄存器可以确认). 所以我们只需要通过 IOMUX 设置复用功能, 通过 GPIO 外设设置好引脚模式即可.
相关的功能可以参考官方文档, 引脚复用的文档位于 `K230_PINOUT_V*.xlsx`.

IOMUX 外设是一个类似 PAD 的结构, 每个引脚通过一个 32 位寄存器设置复用功能, 上拉下拉, 输入输出使能等. 这部分定义我是通过 .dtsi 文件和 C 头文件获得,
也是交给 LLM 来转译成 YAML 定义. `IOMUX.pad(n).set_sel(0)` 即将引脚的模式设置为对应的 GPIO.

GPIO 外设来自 [DW_apb_gpio](https://www.synopsys.com/dw/ipdir.php?c=DW_apb_gpio), 熟悉 Verilog 等 HDL 的朋友一看文档就知道这个是一个最多
4 端口的可配置 GPIO. 有若干配置寄存器可以获取外设的初始参数:

```
GPIO0 config_reg1: num_ports=1
GPIO0 config_reg2: len(PA)=32
GPIO1 config_reg1: num_ports=2
GPIO1 config_reg2: len(PA)=32 len(PB)=8
```

一共 32 + 32 + 8 = 72 个引脚. 分两个 GPIO 控制器, 其中 GPIO1 控制器有两个 PORT. 可以完美适配 chiptool clusture/array 的定义方法.

```rust
fn blinky() {
    // RGB LED of LCKFB
    // - R: GPIO62
    // - G: GPIO20
    // - B: GPIO63
    use pac::{GPIO0, GPIO1, IOMUX};

    IOMUX.pad(20).modify(|w| w.set_sel(0)); // function = GPIOx
    IOMUX.pad(62).modify(|w| w.set_sel(0));
    IOMUX.pad(63).modify(|w| w.set_sel(0));

    GPIO0.swport(0).ddr().modify(|w| *w |= 1 << 20); // output
    GPIO1.swport(0).ddr().modify(|w| *w |= 1 << 30);
    GPIO1.swport(0).ddr().modify(|w| *w |= 1 << 31);

    loop {
        GPIO0.swport(0).dr().modify(|w| *w ^= 1 << 20);
        // GPIO1.swport(0).dr().modify(|w| *w ^= 1 << 30);
        GPIO1.swport(0).dr().modify(|w| *w ^= 1 << 31);

        riscv::delay::McycleDelay::new(CPU0_CORE_CLK).delay_ms(1000);
    }
}
```

### PWM 蜂鸣器

K230 有 6 个 PWM 输出, 分两个 PWM 控制器. 每个控制器内部是 3 个 PWM 输出通道. 1 个额外的通道 0 负责配置 Reload.
庐山派开发板上的蜂鸣器是通过 PWM1 GPIO43 控制的. PWM 外设的输入时钟是 100MHz, 通过 PWMCFG.SCALE 设置 2^n 分频.

为了使蜂鸣器达到人耳可识别的频率, 一般 PWM 的频率设置在 1KHz 左右. 通过 PWMCFG.SCALE 和 PWMx.CMP 设置周期和占空比.
相关代码如下, 寄存器值计算请参考注释

```rust
fn buzzer() {
    // GPIO43 - PWM1
    use pac::{IOMUX, PWM0};

    // PCLK, PWM use APB clock to program registers as well as to generate waveforms. The default frequency is 100MHz.
    IOMUX.pad(43).modify(|w| {
        w.set_sel(2); // PWM = 2
        w.set_oe(true);
        w.set_ds(7);
    });

    // Calc:
    // scale = 2
    // period = 0x5000
    // freq = 100_000_000 / (1 << 2) / 0x5000  = 1220.7 Hz
    // duty = period / 2 = 0x2800
    PWM0.pwmcfg().modify(|w| {
        w.set_zerocomp(true);
        w.set_scale(2);
    });

    PWM0.pwmcmp(0).write(|w| w.0 = 0x5000);
    let duty = 0x2800;

    PWM0.pwmcmp(2).modify(|w| w.0 = duty);

    // enable
    PWM0.pwmcfg().modify(|w| w.set_enalways(true));
    riscv::delay::McycleDelay::new(CPU0_CORE_CLK).delay_ms(100);

    // disable
    PWM0.pwmcfg().modify(|w| w.set_enalways(false));
    riscv::delay::McycleDelay::new(CPU0_CORE_CLK).delay_ms(100);
}
```

## 一些延展思考 (Some Extended Thoughts)

### Why 裸机?

裸机编程是嵌入式开发的基础, 也是最底层的开发方式. 通过裸机编程, 我们可以更好地理解硬件的工作原理, 以及操作系统的工作原理.
用遍全天下的库, 不如自己写一个.

### Shell?

在裸机环境下, 由于没有操作系统, 也没有标准输入输出, 也没有文件系统, 完整的 Shell 是不可能的. 但是我们可以通过串口, 实现简单的命令行交互.
所需要的只是两个串口函数 `putchar` 和 `getchar`, 以及一个简单的解析器.

[noline] 是一个小巧的 no_std line-editing crate, 可以用于实现简单的命令行交互. 而且它基于 embedded-hal 生态, 可以方便地移植.

通过实现若干 shell 命令, 我们可以实现简单的交互, 例如读写外设寄存器, 读写内存, 打印系统信息等等.

相关实现可以参考 [k230-bare-metal] 仓库.

```shell
K230> help
Available commands:
  help - print this help
  echo <text> - print <text>
  reboot - reboot the system
  mem_read <address> <length> - read memory
  mem_write <address> <u32> - write memory
  tsensor - read temperature sensor
  cpuid - print CPUID
  serialboot - enter serial boot mode
```

### Download?

K230 究其定位, 其实更像是 SBC(单板计算机), 烧录固件往往通过 TF 卡, 在逻辑开发中极为不便, 持续插拔 TF 卡会导致接触不良, 甚至损坏.

联想到 [LiteX] 为 FPGA 软核环境提供了非常方便的 kernel/firmware 加载方式, 通过串口下载固件到某一内存位置(DDR), 甚至可以通过网络下载固件.
我尝试移植了 litex_term 的 UART 下载逻辑. 最终的效果是:

```console
> litex_term /dev/tty.usbmodem56C40035621 --kernel-adr 0x01000000 --kernel ../firmware.img
......
Press Q or ESC to abort boot completely.
sL5DdSMmkekro
[LITEX-TERM] Received firmware download request from the device.
[LITEX-TERM] Uploading ../k230-bare-metal-app/big-core.bin to 0x01000000 (17400 bytes)...
[LITEX-TERM] Upload calibration... failed, switching to --safe mode.
[LITEX-TERM] Upload complete (8.7KB/s).
[LITEX-TERM] Booting the device.
[LITEX-TERM] Done.
Jumping to 0x01000000...
```

非常方便, 有机会单独额外介绍. 需要注意 I-Cache 和 D-Cache 的状态, 本文编写时我选择彻底关闭 I-Cache 和 D-Cache.

### 跳转大核

前面说道, CPU1(大核) 的启动是由 CPU0(小核) 控制的. 具体启动逻辑也很简单, 设置复位向量并复位 CPU1 即可:

```rust
unsafe {
    ptr::write_volatile(0x91102104 as *mut u32, jump_addr as u32);
    ptr::write_volatile(0x9110100c as *mut u32, 0x10001000);
    ptr::write_volatile(0x9110100c as *mut u32, 0x10001);
    ptr::write_volatile(0x9110100c as *mut u32, 0x10000);
}
```

为了方便开发测试, 我把跳转大核也做成了 shell 命令. 通过 UART0 输入 `jumpbig 0x01000000` 即可跳转大核执行内存区域代码.
尝试 dump 大核寄存器信息, 可以看到启动信息:

```text
Rust 2nd stage on CPU1
mstatus: 0000000a00001900
mie: 0000000000000000
mip: 0000000000000000
misa: 8000000000b4112f
  RV64ABCDFIMSUVX
mvendorid: 5b7
marchid: 8000000009140d00
mhartid: 0
cpuid: 09140b0d 10050000 260c0001
```

这里 `RV64ABCDFIMSUVX` 中的 `V` 表示支持 RVV 向量指令集, K230 是异构双核, 小核不支持 RVV. 可以证明我们的代码成功跳转到了大核.

当然有个搞笑的, `mhartid` 是 0, 说明 K230 并没有满足 RISC-V 规范给不同的 hart 分配不同的 ID. 这个在实际开发中是需要注意的.
只能通过 misc CSR 来区分不同的 hart.

## 结语 (Conclusion)

通过此次在 K230 芯片 上的 Rust 裸机嵌入式开发，我们深入探索了 MPU 与 MCU 在启动方式和使用模式上的区别，掌握了使用 Rust 进行 MPU 芯片裸机开发的关键步骤，包括启动流程、固件格式解析、中断和外设的初始化等。实践中，我们成功实现了 UART 调试输出、GPIO 点灯、PWM 蜂鸣器等功能，加深了对 K230 底层启动机制和硬件特性的理解。这些成果为日后在 K230 以及其他 RISC-V 芯片上开展更复杂的嵌入式开发奠定了坚实的基础。展望未来，我们可以进一步完善外设驱动，探索多核协同工作、RVV 向量指令的应用，以及结合 Rust 生态构建高效、安全的嵌入式系统，为 RISC-V 开源社区贡献更多力量。(由 GPT 总结)

[CanMV]: https://github.com/kendryte/canmv_k230
[OpenMV]: https://openmv.io/
[RT-Thread]: https://github.com/RT-Thread/rt-thread
[k230-bare-metal]: https://github.com/andelf/k230-bare-metal
[noline]: https://docs.rs/noline/latest/noline/
[LiteX]: https://github.com/enjoy-digital/litex
[chiptool]: https://github.com/embassy-rs/chiptool
[yaml2pac]: https://github.com/embedded-drivers/yaml2pac
