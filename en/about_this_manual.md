### About this manual

This manual describes the ARM® architecture v8, ARMv8. The architecture describes the operation of an
ARMv8-A Processing element (PE), and this manual includes descriptions of:

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

This manual gives the assembler syntax for the instructions it describes, meaning that it describes instructions in
textual form. However, this manual is not a tutorial for ARM assembler language, nor does it describe ARM
assembler language, except at a very basic level. To make effective use of ARM assembler language, read the
documentation supplied with the assembler being used.

This manual is organized into parts:

**Part A**
Provides an introduction to the ARMv8-A architecture, and an overview of the AArch64 and AArch32 Execution states.

