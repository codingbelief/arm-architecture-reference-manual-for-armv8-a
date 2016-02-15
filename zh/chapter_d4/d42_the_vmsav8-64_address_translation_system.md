# D4.2 The VMSAv8-64 address translation system

[`英文版`](../../en/chapter_d4/d42_the_vmsav8-64_address_translation_system.html)

This section describes the VMSAv8-64 address translation system, that maps VAs to PAs. Related to this:
 * [VMSAv8-64 translation table format descriptors on page D4-1695](#) describes the translation table entries.
 * [Access controls and memory region attributes on page D4-1704](#) describes the attributes that are held in the translation table entries, including how different attributes can interact.
 * [Translation Lookaside Buffers (TLBs) on page D4-1729](#) describes the caching of translation table lookups in TLBs, and the architected instructions for maintaining TLBs.
 * [AArch64 Address translation examples on page J7-5480](#) gives detailed descriptions of typical examples of translating a VA to a final PA, and obtaining the memory attributes of that PA.

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
