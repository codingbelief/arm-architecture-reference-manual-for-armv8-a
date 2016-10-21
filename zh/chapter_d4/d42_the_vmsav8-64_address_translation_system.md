# D4.2 VMSAv8-64 地址转换系统 (address translation system)

[`英文版`](../../zh/chapter_d4/d42_the_vmsav8-64_address_translation_system.html)

此章节主要描述将 VAs 映射到 PAs 的 VMSAv8-64 地址转换系统，包括以下内容：

 * [VMSAv8-64 translation table format descriptors](#) 介绍了 translation table entries
 * [Access controls and memory region attributes](#) 介绍了 translation table entries 中的 attributes
 * [Translation Lookaside Buffers (TLBs)](#) 介绍了 TLB 在 translation table lookup 过程中的 caching 用途以及操作 TLB 相关的指令
 * [AArch64 Address translation examples](#) 则通过详细的例子描述了了 VA 到 PA 的转换以及 PA 的 memory attributes 的相关处理

以下的几个小节详细的描述了 VMSAv8-64 address translation system:
 * [About the VMSAv8-64 address translation system](#).
 * [Controlling address translation stages](#).
 * [Memory translation granule size](#).
 * [Translation tables and the translation process](#).
 * [Overview of the VMSAv8-64 address translation stages](#).
 * [The VMSAv8-64 translation table format](#).
 * [The algorithm for finding the translation table entries](#).
 * [The effects of disabling a stage of address translation](#).
 * [The implemented Exception levels and the resulting translation stages and regimes ](#).
 * [Pseudocode description of VMSAv8-64 address translation](#).
 * [Address translation instructions](#).
