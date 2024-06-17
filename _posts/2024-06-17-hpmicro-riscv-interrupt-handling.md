---
layout: post
title: "HPM RISC-V MCU 中断处理简介: 直接地址模式和向量模式 - Interrupt Handling for HPMicro RISC-V MCU / Andes RISC-V"
date: 2024-06-17 15:30 +0800
author: andelf
tags:
  - embedded
  - rust
  - riscv
usemathjax: true
toc: true
published: true
---

Interrupt Handling for HPMicro RISC-V MCU(Andes RISC-V IP Core) - Direct Address Mode and Vector Mode.

最近在折腾 HPMicro 的 RISC-V 系列 MCU 的 Rust 支持 [hpm-hal], 由于需要处理中断, 所以对其中断控制器的工作原理有了一些了解.
虽然 HPMicro 的文档相对于国内其他厂商已经很出色了, 但对于一些细节还是有些模糊, 比如中断的向量模式和具体中断软件处理步骤.
虽然阅读 [hpm_sdk] 代码可以了解到一些细节, 但很容易迷失在条件编译的海洋中.

这里以 HPM5300EVK 为例, MCU 为 HPM5361, IP Core 为 Andes D25(F). HPM6xxx 系列的 IP Core 为 Andes D45, 但是中断控制器的实现是一样的.

- 本文不涉及多核情况. 每个核心各自有一个 PLIC,其工作机制无本质区别.
- 本文不涉及 Supervisor / User 模式, 仅讨论 Machine 模式下的中断处理.
- 本文混合使用 HPM RISC-V MCU 和 Andes IP Core 两个名词, 对于中断处理来说, 他们是通用的.
- 本文使用 [hpm-metapac] 和 [hpm-hal] 作为代码示例.

## 基础介绍

RISC-V 的中断分为核心本地中断(Core Local Interrupts, 即各种缩写中 "CLxxx" 的由来)和外部中断(External Interrupts).
异常也是一种中断, 但是异常是由指令执行引起的. 异常和中断通过 `mcause` 寄存器的最高位来区分. 通过 `mstatus` 和 `mie` 寄存器开启和关闭中断.

### 异常

我们掏出 [riscv-rt] 源码看看具体定义:

```rust
extern "C" {
    fn InstructionMisaligned(trap_frame: &TrapFrame);
    fn InstructionFault(trap_frame: &TrapFrame);
    fn IllegalInstruction(trap_frame: &TrapFrame);
    fn Breakpoint(trap_frame: &TrapFrame);
    fn LoadMisaligned(trap_frame: &TrapFrame);
    fn LoadFault(trap_frame: &TrapFrame);
    fn StoreMisaligned(trap_frame: &TrapFrame);
    fn StoreFault(trap_frame: &TrapFrame);
    fn UserEnvCall(trap_frame: &TrapFrame);
    fn SupervisorEnvCall(trap_frame: &TrapFrame);
    fn MachineEnvCall(trap_frame: &TrapFrame);
    fn InstructionPageFault(trap_frame: &TrapFrame);
    fn LoadPageFault(trap_frame: &TrapFrame);
    fn StorePageFault(trap_frame: &TrapFrame);
}

#[doc(hidden)]
#[no_mangle]
pub static __EXCEPTIONS: [Option<unsafe extern "C" fn(&TrapFrame)>; 16] = [
    Some(InstructionMisaligned),
    Some(InstructionFault),
    Some(IllegalInstruction),
    Some(Breakpoint),
    Some(LoadMisaligned),
    Some(LoadFault),
    Some(StoreMisaligned),
    Some(StoreFault),
    Some(UserEnvCall),
    Some(SupervisorEnvCall),
    None,
    Some(MachineEnvCall),
    Some(InstructionPageFault),
    Some(LoadPageFault),
    None,
    Some(StorePageFault),
];
```

[riscv-rt] 通过静态数组 `__EXCEPTIONS` 定义了异常处理函数列表, 通过 `mcause` 寄存器的值来索引.
`TrapFrame` 是一个结构体, 用于保存异常发生时的寄存器状态. 由汇编代码保存后压栈传递.
所有函数由链接脚本提供了一个 `loop {}` 死循环的默认实现, 用户可以在代码中覆盖.

### 中断

```rust
extern "C" {
    fn SupervisorSoft();
    fn MachineSoft();
    fn SupervisorTimer();
    fn MachineTimer();
    fn SupervisorExternal();
    fn MachineExternal();
}

#[doc(hidden)]
#[no_mangle]
pub static __INTERRUPTS: [Option<unsafe extern "C" fn()>; 12] = [
    None,
    Some(SupervisorSoft),
    None,
    Some(MachineSoft),
    None,
    Some(SupervisorTimer),
    None,
    Some(MachineTimer),
    None,
    Some(SupervisorExternal),
    None,
    Some(MachineExternal),
];
```

如上代码是中断处理函数的定义, 处理方式一致, 也是通过静态数组 `__INTERRUPTS` 索引. 但是中断处理函数不需要传递 `TrapFrame`, 因为中断发生时,
由中断处理函数来保存寄存器和恢复状态.

其中我们需要关注的是 `MachineExternal` 函数, 它负责在机器模式下处理外部中断(即 MCU 的外设中断).
当发生外部中断时, 在 `MachineExternal` 函数中, 需要先读取 `PLIC` 的中断挂起状态, 确定具体的中断源, 然后调用相应的中断处理函数进行处理.

### 中断入口

[riscv-rt] 默认使用直接模式(Direct Mode)处理中断, 即将 `mtvec` 寄存器设置为统一的中断入口函数地址。
在这种模式下, 发生中断时会直接跳转到该函数执行. 中断入口函数需要读取 `mcause` 寄存器的值, 判断是异常还是中断, 然后调用相应的异常或中断处理逻辑.
具体实现中, 入口地址为汇编代码, 负责调用 Rust 函数, 并执行 mret 指令返回.

(这部分实现 [riscv-rt] 经常修改, 今天汇编改 Rust, 明天 Rust 改宏, 大后天可能又换个位置..., 实际上区别不大, 依然很难用)

而 Rust 部分比较直观:

```rust
pub unsafe extern "C" fn start_trap_rust(trap_frame: *const TrapFrame) {
    extern "C" {
        fn ExceptionHandler(trap_frame: &TrapFrame);
        fn DefaultHandler();
    }

    let cause = xcause::read();
    let code = cause.code();

    if cause.is_exception() {
        let trap_frame = &*trap_frame;
        if code < __EXCEPTIONS.len() {
            let h = &__EXCEPTIONS[code];
            if let Some(handler) = h {
                handler(trap_frame);
            } else {
                ExceptionHandler(trap_frame);
            }
        } else {
            ExceptionHandler(trap_frame);
        }
        ExceptionHandler(trap_frame)
    } else if code < __INTERRUPTS.len() {
        let h = &__INTERRUPTS[code];
        if let Some(handler) = h {
            handler();
        } else {
            DefaultHandler();
        }
    } else {
        DefaultHandler();
    }
}
```

这样当我们需要处理外部中断时候, 只需要覆盖 `MachineExternal` 函数即可, `__INTERRUPTS` 通过链接符号获得函数入口地址.

```rust
#[no_mangle]
extern "C" fn MachineExternal() {
    let irq_no = pac::PLIC.claim();

    // handle irq_no here!

    pac::PLIC.complete(irq_no);
}
```

一般情况下, [svd2rust] 生成的 pac crate 会自带一个 `__EXTERNAL_INTERRUPTS` 表, 我们在 `MachineExternal` 中读取当前外部中断号, 然后跳转处理即可.

部分 RISC-V 核心实现可能会有一些特殊的处理, 以具体型号参考文档为准.

## HPM RISC-V MCU 中断处理 - 直接地址模式

这里介绍 HPM RISC-V MCU 中断处理的两种模式, 包括传统的直接地址模式和 Andes IP Core 特有的向量模式.

[riscv-rt] 支持通过编译选项选择中断处理模式, 但向量模式实现极不通用, 直接地址模式是最常见的, 也是几乎被所有 RISC-V 实现支持的.

直接地址模式大致流程如下:

- 中断模式启用:
  - `mstatus` 中 `MIE` 位用于开启全局中断
  - `mie` 中 `MEXT` 位用于开启外部中断, `MTIMER` 位用于开启 MTIME 中断...
  - `mtvec` 写入中断处理函数地址, 低位置 0(无影响)
- 中断处理入口 - [riscv-rt] 提供
  - 从 `mcuase` 可以获得当前是异常还是中断, 中断号等信息
  - 从静态数组中读取对应的处理函数地址, 调用中断处理函数中处理具体的中断
  - `mret` 返回
- 外部中断处理函数 - 链接符号覆盖
  - 读取 PLIC, 获取当前中断号
  - 处理中断
  - 完成中断, 通知 PLIC (PLIC.claim)

### GPIO 外设 - 外部中断

对于外部中断来说, 处理 `MachineExternal` 即可.

需要注意的是 `mtvec` 的设置在 [riscv-rt] 中完成, 通过 `xtvec::write(_start_trap as usize, xTrapMode::Direct);` 写入函数地址和中断模式即可.
但 Andes RISC-V IP Core 的 mtvec 低位地址无效, 它使用额外的自定义 CSR 来选择中断模式. 这里 `mtvec` 只用于写入函数入口地址.

在 `mtvec` 设置后, 还需要额外设置 `mstatus`, `mie` CSR 寄存器, 开启全局中断, 和外部中使能.

```rust
unsafe {
    riscv::register::mstatus::set_mie(); // enable global interrupt
    riscv::register::mie::set_mext(); // enbale external interrupt
}
```

这里以 PA09 GPIO0 中断为例(开发板板载按钮), 设置外设中断:

```rust
// 省略 GPIO 配置
pac::GPIO0.pl(0).set().write(|w| w.set_irq_pol(1 << 9)); // falling edge
pac::GPIO0.tp(0).set().write(|w| w.set_irq_type(1 << 9)); // edge trigger
pac::GPIO0.ie(0).set().write(|w| w.set_irq_en(1 << 9)); // enable interrupt
```

然后针对具体的外设中断号通过 PLIC 启用中断:

```rust
unsafe {
    hal::interrupt::GPIO0_A.set_priority(Priority::P1); // PLIC.priority
    hal::interrupt::GPIO0_A.enable(); // PLIC.targetint[0].inten
}
```

这里我们直接覆盖 `MachineExternal` 函数即可.

```rust
#[no_mangle]
extern "C" fn MachineExternal() {
    let claim = pac::PLIC.claim(); // 获取当前 interrupt id

    defmt::info!("claim = {}", claim);

    if claim == hal::interrupt::GPIO0_A.number() {
        // GPIO0_A();
        // write 1 to clear
        pac::GPIO0.if_(0).value().write(|w| w.set_irq_flag(1 << 9)); // 清除对应外设的中断标志
    }

    pac::PLIC.complete(claim); // 通知 PLIC 处理完毕
}
```

这样就完成了对 PA09 GPIO0 的中断处理.

### MCHTMR - MTIME 中断

以上举例的是 MCU 的外设中断, 这里再介绍下 MTIME, 即 MCHTMR 中断的处理. 假定 `mstatus::set_mie()` 已经设置, 即全局中断已经开启.

```rust
let val = pac::MCHTMR.mtime().read();
let next = val + 24_000_000; // 24Mhz default

pac::MCHTMR.mtimecmp().write_value(next); // + 1s

unsafe {
    riscv::register::mie::set_mtimer();
}
```

这里我们直接设置了一个 1 秒之后的中断. 然后在 `MachineTimer` 中处理即可.

```rust
#[no_mangle]
extern "C" fn MachineTimer() {
    // // disable mtime interrupt
    //  unsafe {
    //        riscv::register::mie::clear_mtimer();
    //}

    let val = pac::MCHTMR.mtime().read();
    let next = val + 24_000_000; // 24Mhz

    pac::MCHTMR.mtimecmp().write_value(next);
}
```

中断处理函数 `MachineTimer` 中可以直接 `mie::clear_mtimer()` 关闭 MTIME 中断, 也可以设置比较寄存器, 设置下次中断时间.

## HPM RISC-V MCU 中断处理 - 向量模式

中断向量模式在 HPM 官方 Datasheet 中介绍较为简略, 同时还需要参考 Andes RISC-V IP Core 的[文档](http://www.andestech.com/en/products-solutions/product-documentation/).

- AndeStar V5 Platform-Level Interrupt Controller Specification (PLIC IP Core 文档)
- AndeStar V5 System Privileged Architecture and CSR (CSR 文档)

经过相关试验, 得到如下结论:

- 中断模式启用:
  - `mstatus` 中 `MIE` 位用于开启全局中断
  - `mie` 中 `MEXT` 位用于开启外部中断, `MTIMER` 位用于开启 MTIME 中断...
  - `mtvec` 写入中断表地址, `mtvec` 低 2 位无效 - **与标准 RISC-V 行为不一致**
  - 自定义 CSR `mmisc_ctl`(0x7D0) 和 PLIC 设置位 `PLIC.FEATURE.VECTORED` 用于选择向量模式
  - 中断表地址必须位于 FLASH(XPI) 或 ILM, **无法放在 DLM**, 编写自定义链接脚本时候需要注意
- 中断表
  - 中断表索引为中断号, 4 字节内存地址表, 必须 4 字节对齐
  - 外设中断(外部中断)编号从 1 开始
  - 中断表索引 0 为异常处理和 Core Local 中断, 即上面提到的 `__EXCEPTIONS` 和 `__INTERRUPTS`
  - 中断表 `mtvec[N]` 长 1024, 对于 Supervisor 模式和 User 模式, 中断表为 `mtvec[1024+N]`, `mtvec[2048+N]`
- 中断处理函数 - 外部中断
  - **此模式下 `PLIC.CLAIM` 寄存器值无效**, 需要当前函数名获得, 或者通过 `mcause` 读取
  - 处理中断
  - 写入中断号到 `PLIC.CLAIM` 通知 PLIC 处理完毕
  - `mret` 返回
- 中断处理函数 - Core Local 中断
  - 在向量模式下, 内部中断和异常处理函数位于向量表 0 位置
  - 通过读取 `mcause` 寄存器值, 获取异常或中断原因, 然后调用对应的异常或中断处理函数
  - `mret` 返回

### Patch [riscv-rt] `_setup_interrupts`

[riscv-rt] 中的中断处理默认是直接地址模式, 通过编译选项 `v-trap` 可以切换到向量中断处理模式,
但是实现无法兼容 HPM RISC-V MCU 的向量中断模式. 传统 RISC-V 标准实现的向量中断模式,
指的是跳转指令的向量表, 即中断向量每个中断号对应一条 jump 指令, 跳转到对应的中断处理函数.

而 Andes IP Core 的中断向量是一个内存地址表, 每个中断号对应一个函数的内存地址.
这样的好处是直接跳转, 没有跳转指令的内存偏移限制.

万幸是 [riscv-rt]  提供了扩展点, 可以通过覆盖 `_setup_interrupts` 函数来实现自定义中断处理相关设置.

```rust
#[no_mangle]
pub unsafe extern "Rust" fn _setup_interrupts() {
    extern "C" {
        static __VECTORED_INTERRUPTS: [u32; 1];
    }

    let vector_addr = __VECTORED_INTERRUPTS.as_ptr() as u32;
    // FIXME: TrapMode is ignored in mtvec, it's set in CSR_MMISC_CTL
    riscv::register::mtvec::write(vector_addr as usize, riscv::register::mtvec::TrapMode::Direct);

    // Enable vectored external PLIC interrupt
    // CSR_MMISC_CTL = 0x7D0
    unsafe {
        asm!("csrsi 0x7D0, 2");
        pac::PLIC.feature().modify(|w| w.set_vectored(true));
    }
}
```

`__VECTORED_INTERRUPTS` 是在 [hpm-metapac] 中定义的向量模式中断表, 通过 `extern` 方式从链接器过程获取地址.

此时我们可能需要自定义链接脚本, 将 [riscv-rt] 中的额外无效符号丢弃. 不过目前暂时不需要.

### 中断处理函数 - 内部中断和异常

考虑到向量表 0 位置用于处理处理器内部中断和异常, 我们希望能复用 [riscv-rt] 中的异常处理函数定义表. 在 `hpm-metapac` 的实现中,
我定义向量表第一个位置为 `CORE_LOCAL` 中断处理函数(通过代码生成工具自动处理):

```rust
#[link_section = ".vector_table.interrupts"]
#[no_mangle]
pub static __VECTORED_INTERRUPTS: [Vector; 73] = [
    Vector { _handler: CORE_LOCAL },
    Vector { _handler: GPIO0_A },
    Vector { _handler: GPIO0_B },
    Vector { _handler: GPIO0_X },
    Vector { _handler: GPIO0_Y },
    Vector { _handler: GPTMR0 },
    Vector { _handler: GPTMR1 },
    // ......
}
```

这样就可以通过链接符号直接添加内部中断处理函数:

```rust
#[no_mangle]
unsafe extern "riscv-interrupt-m" fn CORE_LOCAL() {
    // 使用这样的方式, 可以链接非 `pub` 向量表
    extern "C" {
        static __INTERRUPTS: [Option<unsafe extern "C" fn()>; 12];
    }

    let cause = riscv::register::mcause::read();
    let code = cause.code();

    defmt::info!("mcause = 0x{:08x}", cause.bits());
    if cause.is_exception() {
        loop {} // let it crash for now
    } else if code < __INTERRUPTS.len() {
        let h = &__INTERRUPTS[code];
        if let Some(handler) = h {
            handler();
        } else {
            DefaultHandler();
        }
    } else {
        DefaultHandler();
    }
}
#[allow(non_snake_case)]
#[no_mangle]
fn DefaultHandler() {
    loop {}
}
```

由于没有了 [riscv-rt] 的中断入口函数, 我们需要自己处理寄存器压栈和恢复, 以及中断结束的 `mret` 指令.
`extern "riscv-interrupt-m" fn` 是一个特殊的 ABI, 它会自动处理所有寄存器的保存和恢复,并在中断处理函数返回时执行 `mret` 指令.
需要 Nightly Rust 和 `#![feature(abi_riscv_interrupt)]` feature 启用.

使用这种方式, 可以在不 fork [riscv-rt] 的情况下, 实现与传统 [riscv-rt] 中断处理方式类似的功能.

这里我踩了一个不小的坑, 当开启向量模式后, `PLIC.CLAIM` 不再提供中断号, 但中断处理函数依然需要写入它来通知 `PLIC` 处理完毕.
因此, 需要通过读取 `mcause` 寄存器获取中断号, 然后写入 `PLIC.CLAIM` 寄存器来通知 `PLIC` 中断已经处理完毕.

### 中断处理函数 - 外部中断

```rust
#[no_mangle]
unsafe extern "riscv-interrupt-m" fn GPIO0_A() {
    // let claim = pac::PLIC.claim(); // WRONG!!!
    let mcause = mcause::read().bits();

    defmt::info!("button pressed!");

    // write 1 to clear
    pac::GPIO0.if_(0).value().write(|w| w.set_irq_flag(1 << 9));

    compiler_fence(core::sync::atomic::Ordering::SeqCst);
    pac::PLIC.complete(mcause as u16);
}
```

## 总结

本文中对 `PLIC` 操作的实现位于 [hpm-hal] 的 [src/internal/interrupt.rs](https://github.com/hpmicro-rs/hpm-hal/blob/master/src/internal/interrupt.rs).

[riscv-rt] 的实现不够通用, 但疯狂打 patch 还是能将就的. 如果我已经自定义了向量表, 中断处理, 链接脚本, 那么原本属于它的只剩下几句 `.data` `.bss` 和 FPU 初始化汇编了. 并不是很好的方式.

向量模式中断总体实现更简洁明了, 少一层跳转, 其中 `extern "riscv-interrupt-m" fn` 可以通过过程宏的方式处理, 例如 `#[interrupt]`.

本文适用于其他 Andes IP Core 的 RISC-V MCU, 但是具体实现细节请参考具体型号的文档.

本文适用于其他语言. 但是需要注意 ABI 和寄存器的处理.

欢迎关注 [hpm-hal] 的进展, 以及 [hpm-metapac] 的实现 [hpm-data].

[hpm_sdk]: https://github.com/hpmicro/hpm_sdk
[hpm-hal]: https://github.com/hpmicro-rs/hpm-hal
[riscv-rt]: https://docs.rs/crate/riscv-rt/latest/source/src/lib.rs
[svd2rust]: https://github.com/rust-embedded/svd2rust
[hpm-metapac]: https://github.com/hpmicro-rs/hpm-metapac
[hpm-data]: https://github.com/andelf/hpm-data
