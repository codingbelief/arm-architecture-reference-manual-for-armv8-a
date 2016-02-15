### 关于本参考手册

[`英文版`](../../en/about_this_manual.html)

This manual describes the ARM® architecture v8, ARMv8. The architecture describes the operation of an ARMv8-A Processing element (PE), and this manual includes descriptions of:

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
 * The Advanced SIMD and floating-point instructions, that provide high-performance:
 * 高性能的 SIMD 和浮点指令：
    - 支持单精度和双精度浮点数操作。
    - 双精度、单精度和半精度浮点数转换。
    - 三种指令集都支持整形、单精度浮点数向量操作。
    - 在 A64 运行态下，支持双精度浮点数向量操作。
 * The security model, that provides two security states to support secure applications.
 * 安全模式 (security model)，
 * The virtualization model, that support the virtualization of Non-secure operation.
 * The Debug architecture, that provides software access to debug features.

This manual gives the assembler syntax for the instructions it describes, meaning that it describes instructions in
textual form. However, this manual is not a tutorial for ARM assembler language, nor does it describe ARM
assembler language, except at a very basic level. To make effective use of ARM assembler language, read the
documentation supplied with the assembler being used.

This manual is organized into parts:

**Part A**  
Provides an introduction to the ARMv8-A architecture, and an overview of the AArch64 and AArch32 Execution states.

**Part B**
Describes the application level view of the AArch64 Execution state, meaning the view from EL0. It describes the application level view of the programmers’ model and the memory model.

**Part C**  
Describes the A64 instruction set, that is available in the AArch64 Execution state. The descriptions for each instruction also include the precise effects of each instruction when executed at EL0, described as unprivileged execution, including any restrictions on its use, and how the effects of the instruction differ at higher Exception levels. This information is of primary importance to authors and users of compilers, assemblers, and other programs that generate ARM machine code.

**Part D**  
Describes the system level view of the AArch64 Execution state. It includes details of the System registers, most of which are not accessible from EL0, and the system level view of the programmers’  model and the memory model. This part includes the description of self-hosted debug.

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

