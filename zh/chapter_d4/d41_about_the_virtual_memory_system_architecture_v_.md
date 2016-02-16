
# D4.1 虚拟内存系统架构 (VMSA) 简介

[`英文版`](../../en/chapter_d4/d41_about_the_virtual_memory_system_architecture_v_.html)

本小节描述了 AArch64 运行态下的 PE 中的虚拟内存系统架构 (VMSA)，
即 VMSAv8-64 (Virtual Memory System Architecture on AArch64)。

VMSA 包含内存管理单元，即 MMU (Memory Management Unit)。MMU 在系统中为 PE 的内存操作提供地址转换，访问权限控制以及内存属性的设定和校验功能。

MMU 中的地址转换过程，是将 PE 发起的内存访问的虚拟地址 (VA, Virtual Address)
映射到物理内存设备的真实物理地址 (PA, Physical Address) 上。这一转换过程，会因 PE 所在的 Exception level 和所处的 Security state 的不同而不同，具体的差异可以参考 Figure D4-1。

![](figure_d4_1.png)

VMSAv8-64 支持虚拟地址标签功能 (Virtual Address Tagging)，可以将虚拟地址的若干位作为标签，用于其他用途。用于虚拟地址标签的比特位不参与地址转换过程，后续的[D4.1.1 AArch64 下的地址标签 (Address Tagging) ](d41_1_address_tagging_in_aarch64_state.md)小节会对此进行详细的描述。

本章节后续的内容将对实现了所有 Exception levels 的系统上的 VMSAv8-64 进行详细的介绍。对于只实现了部分 Exception levels 的系统上的 VMSAv8-64，可以参考
[The implemented Exception levels and the resulting translation stages and regimes on page D4-1679](todo.md) 
章节的描述
