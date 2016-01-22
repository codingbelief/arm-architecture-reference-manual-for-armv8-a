># D4.1 About the Virtual Memory System Architecture (VMSA)

# D4.1 VMSA 简介

> This chapter describes the Virtual Memory System Architecture (VMSA) 
 > that applies to a PE executing in AArch64 state.
> This is VMSAv8-64, as defined in [ARMv8 VMSA naming on page D4-1643](todo.md).

此小节描述了 AArch64 运行态下的 PE 中的虚拟内存系统架构，
即VMSAv8-64 (Virtual Memory System Architecture on AArch64)。
(TODO: 添加 PE 的解释)

> A VMSA provides a Memory Management Unit (MMU), 
 > that controls address translation, access permissions,
 > and memory attribute determination and checking, 
 > for memory accesses made by the PE.

VMSA 实现了内存管理单元，即MMU。MMU在系统中为PE的内存操作提供地址转换，
访问权限控制以及内存属性的设定和校验功能。

> The process of address translation maps the virtual addresses (VAs) used by 
 > PE onto the physical addresses (PAs) of the physical memory system. 
> These translations are defined independently for different Exception levels
 > and Security states, and Figure D4-1 shows.

MMU 中的地址转换过程，是将 PE 发起的内存访问的虚拟地址 (VAs)
映射到物理内存设备的真实物理地址上。这一转换过程，会因 PE 所在的
Exception levels 和所处的 Security states 的不同而不同，具体的差异可以参考 Figure D4-1。
