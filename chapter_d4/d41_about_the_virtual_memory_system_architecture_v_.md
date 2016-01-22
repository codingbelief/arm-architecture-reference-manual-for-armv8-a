># D4.1 About the Virtual Memory System Architecture (VMSA)

# D4.1 VMSA 简介

> This chapter describes the Virtual Memory System Architecture (VMSA) 
 > that applies to a PE executing in AArch64 state.
> This is VMSAv8-64, as defined in [ARMv8 VMSA naming on page D4-1643](todo.md).

此小节描述了 AArch64 运行态下的 PE 中的虚拟内存系统架构，
即VMSAv8-64 (Virtual Memory System Architecture on AArch64)。

> A VMSA provides a Memory Management Unit (MMU), 
 > that controls address translation, access permissions,
 > and memory attribute determination and checking, 
 > for memory accesses made by the PE.

VMSA 为 PE 的内存访问提供内存管理单元实现，即MMU。MMU在系统中主要负责地址转换，访问权限控制
以及内存属性的设定和校验。

> The process of address translation maps the virtual addresses (VAs) used by 
 > PE onto the physical addresses (PAs) of the physical memory system. 
> These translations are defined independently for different Exception levels
 > and Security states, and Figure D4-1 shows.

