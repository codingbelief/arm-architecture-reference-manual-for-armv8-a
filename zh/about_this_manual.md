### 关于本参考手册

[`英文版`](../../en/about_this_manual.html)


本手册主要描述了 ARMv8 体系结构。ARMv8 体系结构主要描述了 ARMv8-A 处理单元 (PE，Processing element) 的运行机制，包括以下方面内容：

 * AArch64 和 AArch32 两个运行态。
 * 三种指令集:
    - 在 AArch32 运行态下, 支持兼容旧架构的 A32 和 T32 指令集.
    - 在 AArch64 运行态下, 执行 A64 指令集.
 * 当前 Exception 等级， 安全状态和运行态的不同对 PE 行为的影响。
 * Exception 模型 (Exception model)。
 * 支持 AArch64 和 AArch32 运行态切换的内部交互模型 (interprocessing model)。
 * 定义内存端顺 (memory ordering) 和管理 (memory management) 的内存模型 (memory model)。本手册中，仅描述定义了虚拟内存系统架构 (VMSA) 的 ARMv8-A 架构的内存模型。
 * 编程模型 (programmers’ model)，主要描述用于控制 PE 和内存系统，以及提供相关状态信息的系统寄存器 (System registers) 接口。
 * 高性能的 SIMD 和浮点指令：
    - 支持单精度和双精度浮点数操作。
    - 双精度、单精度和半精度浮点数转换。
    - 三种指令集都支持整形、单精度浮点数向量操作。
    - 在 A64 运行态下，支持双精度浮点数向量操作。
 * 安全模式 (security model)，提供了 secure 和 Non-secure 两种安全状态，用于支持安全应用场景。
 * 虚拟化模型 (virtualization model)，支持 Non-secure 操作的虚拟化。
 * 调式架构，提供了可软件控制的调试特性。

本手册中的指令是用文本形式的汇编语法来描述的，但是本手册并不是 ARM 汇编语言教程，手册中只简单的介绍了汇编语言最基础的部分，更多汇编语言的细节，需要参考实际所使用的汇编器的相关文档。

本手册的内容分为以下几个部分：


**Part A**  
此部分内容主要对 ARMv8-A 体系结构进行简单介绍，概括性的描述了 AArch64 和 AArch32 运行态，简要描述其所支持的数据类型等。

**Part B**  
此部分内容从应用层面 (application level view) 描述 AArch64 运行态 (也可以说在 EL0 层次上进行描述)，它主要介绍了应用层下的编程模型和内存模型。

**Part C**  
此部分内容详细介绍了 AArch64 运行态下的 A64 指令集。对于每一个所描述的指令都详细的描述了其在 EL0 下执行 (即非特权模式执行) 时的效果，同时也描述了指令在其他更高级别的 Exception level 下执行时，与 EL0 下执行的差异。这部分内容，对于编译器、汇编器和其他用于生成 ARM 机器码的软件的设计者和使用者都非常关键。

**Part D**  
Describes the system level view of the AArch64 Execution state. It includes details of the System registers, most of which are not accessible from EL0, and the system level view of the programmers’  model and the memory model. This part includes the description of self-hosted debug.
此部分内容从系统层面 (system level view) 描述 AArch64 运行态。它主要描述了系统层下的编程模型和内存模型，同时还详细介绍了 EL0 中无法访问的系统寄存器 (System registers)。

**Part E**  
Describes the application level view of the AArch32 Execution state, meaning the view from the EL0. It describes the application level view of the programmers’ model and the memory model.


> **NOTE:**  
In AArch32 state, execution at EL0 is execution in User mode.


**Part F**  
Describes the T32 and A32 instruction sets, that are available in the AArch32 Execution state. These instruction sets are backwards-compatible with earlier versions of the ARM architecture. This part describes the precise effects of each instruction when executed in User mode, described as unprivileged execution or execution at EL0, including any restrictions on its use, and how the effects
of the instruction differ at higher Exception levels. This information is of primary importance to authors and users of compilers, assemblers, and other programs that generate ARM machine code.

>**NOTE:**  
User mode is the only mode where software execution is unprivileged.


**Part G**  
Describes the system level view of the AArch32 Execution state, that is generally compatible with earlier versions of the ARM architecture. This part includes details of the System registers, most of which are not accessible from EL0, and the conceptual coprocessor interface to those registers. It also describes the system level view of the programmers’ model and the memory model.


**Part H**  
Describes the Debug architecture for external debug. This provides configuration, breakpoint and watchpoint support, and a Debug Communications Channel (DCC) to a debug host.


**Part I**  
Describes additional features of the architecture that are not closely coupled to a processing element (PE), and therefore are accessed through memory-mapped interfaces. Some of these features are OPTIONAL.


**Appendixes**  
Provide additional information. Some appendixes give information that is not part of the ARMv8 architectural requirements. The cover page of each appendix indicates its status.

