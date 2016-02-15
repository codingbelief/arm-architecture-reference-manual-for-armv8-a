### About this manual

[`中文版`](../../zh/about_this_manual.html)

This manual describes the ARM® architecture v8, ARMv8. The architecture describes the operation of an ARMv8-A Processing element (PE), and this manual includes descriptions of:

 * The two Execution states, AArch64 and AArch32.
 * The instruction sets:
    - In AArch32 state, the A32 and T32 instruction sets, that are compatible with earlier versions of the ARM architecture.
    - In AArch64 state, the A64 instruction set.
 * The states that determine how a PE operates, including the current Exception level and Security state, and in AArch32 state the PE mode.
 * The Exception model.
 * The interprocessing model, that supports transitioning between AArch64 state and AArch32 state.
 * The memory model, that defines memory ordering and memory management. This manual covers a single architecture profile, ARMv8-A, that defines a Virtual Memory System Architecture (VMSA).
 * The programmers’ model, and its interfaces to System registers that control most PE and memory system features, and provide status information.
 * The Advanced SIMD and floating-point instructions, that provide high-performance:
    - Single-precision and double-precision floating-point operations.
    - Conversions between double-precision, single-precision, and half-precision floating-point values.
    - Integer, single-precision floating-point, and in A64, double-precision vector operations in all instruction sets.
    - Double-precision floating-point vector operations in the A64 instruction set.
 * The security model, that provides two security states to support secure applications.
 * The virtualization model, that support the virtualization of Non-secure operation.
 * The Debug architecture, that provides software access to debug features.

This manual gives the assembler syntax for the instructions it describes, meaning that it describes instructions in textual form. However, this manual is not a tutorial for ARM assembler language, nor does it describe ARM assembler language, except at a very basic level. To make effective use of ARM assembler language, read the documentation supplied with the assembler being used.

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

