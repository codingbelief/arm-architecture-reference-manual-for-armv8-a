# D4.2 VMSAv8-64 地址转换系统 (address translation system)

[`英文版`](../../zh/chapter_d4/d42_the_vmsav8-64_address_translation_system.html)

此章节主要描述将 VAs 映射到 PAs 的 VMSAv8-64 地址转换系统，包括以下内容：

 * [VMSAv8-64 translation table format descriptors](#) 介绍了 translation table entries
 * [Access controls and memory region attributes](#) 介绍了 translation table entries 中的 attributes
 * [Translation Lookaside Buffers (TLBs)](#) 介绍了 TLB 在 translation table lookup 过程中的 caching 用途以及操作 TLB 相关的指令
 * [AArch64 Address translation examples](#) 则通过详细的例子描述了了 VA 到 PA 的转换以及 PA 的 memory attributes 的相关处理

In this section, the following subsections describe the VMSAv8-64 address translation system:
 * [About the VMSAv8-64 address translation system](#).
 * [Controlling address translation stages on page D4-1645](#).
 * [Memory translation granule size on page D4-1651](#).
 * [Translation tables and the translation process on page D4-1656](#).
 * [Overview of the VMSAv8-64 address translation stages on page D4-1658](#).
 * [The VMSAv8-64 translation table format on page D4-1667](#).
 * [The algorithm for finding the translation table entries on page D4-1674](#).
 * [The effects of disabling a stage of address translation on page D4-1677](#).
 * [The implemented Exception levels and the resulting translation stages and regimes on page D4-1679](#).
 * [Pseudocode description of VMSAv8-64 address translation on page D4-1680](#).
 * [Address translation instructions on page D4-1691](#).
