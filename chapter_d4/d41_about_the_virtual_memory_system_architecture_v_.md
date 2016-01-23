># D4.1 About the Virtual Memory System Architecture (VMSA)

# D4.1 VMSA 简介

> This chapter describes the Virtual Memory System Architecture (VMSA) 
 > that applies to a PE executing in AArch64 state.
> This is VMSAv8-64, as defined in [ARMv8 VMSA naming on page D4-1643](todo.md).

此小节描述了 AArch64 运行态下的 PE 中的虚拟内存系统架构 (VMSA)，
即 VMSAv8-64 (Virtual Memory System Architecture on AArch64)。
(*TODO: 添加 PE 的解释*)

> A VMSA provides a Memory Management Unit (MMU), 
 > that controls address translation, access permissions,
 > and memory attribute determination and checking, 
 > for memory accesses made by the PE.

<!-- VMSA 定义了内存管理单元 这句不知道如何翻译会更贴切 -->
VMSA 定义了内存管理单元，即 MMU。MMU 在系统中为 PE 的内存操作提供地址转换，
访问权限控制以及内存属性的设定和校验功能。

> The process of address translation maps the virtual addresses (VAs) used by 
 > PE onto the physical addresses (PAs) of the physical memory system. 
> These translations are defined independently for different Exception levels
 > and Security states, and Figure D4-1 shows.

MMU 中的地址转换过程，是将 PE 发起的内存访问的虚拟地址 (VAs)
映射到物理内存设备的真实物理地址上。这一转换过程，会因 PE 所在的
Exception levels 和所处的 Security states 的不同而不同，具体的差异可以参考 Figure D4-1。
(*TODO: 简单解释下图中的转换过程*)
![](figure_d4_1.png)

> VMSAv8-64 supports tagging of VAs,
 >as described in [Address tagging in AArch64 state](todo.md).
> As that section describes,
 > this address tagging has no effect on the address translation process.

VMSAv8 支持 VAs 的 tagging，详细的介绍可以参考后续小节。

> The remainder of this chapter gives a full description of VMSAv8-64
 > for an implementation that includes all of the Exception levels.
> [The implemented Exception levels and the resulting translation stages and regimes on
 >page D4-1679](todo.md) describes the differences in the VMSA
 > if some Exception levels are not implemented.
 
本章节后续的内容将对实现了所有 Exception levels 的系统上的 VMSAv8-64 进行详细的介绍。
对于只实现了部分 Exception levels 的系统上的  VMSAv8-64，可以参考
[The implemented Exception levels and the resulting translation stages and regimes on page D4-1679](todo.md) 
章节的描述

# D4.1.1 Address tagging in AArch64 state

In AArch64 state, the ARMv8 architecture supports tagged addresses for data values. In these cases the top eight
bits of the virtual address are ignored when determining:
* Whether the address causes a Translation fault from being out of range if the translation system is enabled.
* Whether the address causes an Address size fault from being out of range if the translation system is not
enabled.
* Whether the address requires invalidation when performing a TLB invalidation instruction by address.


The use of address tags is controlled as follows:

**For addresses using the VMSAv8-64 EL1&0 translation regime**

The value of bit[55] of the VA determines the register bit that controls the use of address tags, as
follows:

| VA[55]==0 | TCR_EL1.TBI0 determines whether address tags are used. If stage 1 translation is enabled, TTBR0_EL1 holds the base address of the translationtables used to translate the address.|
| -- | -- |
| VA[55]==1 | TCR_EL1.TBI1 determines whether address tags are used. If stage 1 translation is enabled, TTBR1_EL1 holds the base address of the translation tables used to translate the address.|


**For addresses using the VMSAv8-64 EL2 translation regime**

TCR_EL2.TBI determines whether address tags are used. If stage 1 translation is enabled,
TTBR0_EL2 holds the base address of the translation tables used to translate the address.


**For addresses using the VMSAv8-64 EL3 translation regime**

TCR_EL3.TBI determines whether address tags are used. If stage 1 translation is enabled,
TTBR0_EL3 holds the base address of the translation tables used to translate the address.


>**NOTE**:

>The TCR_ELx.TBIn bits determine whether address tags are used regardless of whether the corresponding
>translation regime is enabled.


An address tag enable bit also has an effect on the PC value in the following cases:
Any branch or procedure return within the controlled Exception level.
* On taking an exception to the controlled Exception level, regardless of whether this is also the Exception
   level from which the exception was taken.
* On performing an exception return to the controlled Exception level, regardless of whether this is also the
   Exception level from which the exception return was performed.
* Exiting from debug state to the controlled Exception level.

>**NOTE:**

>As an example of what is meant by the controlled Exception level, TCR_EL2.TBI controls this effect for:
>* A branch or procedure return within EL2.
>* Taking an exception to EL2.
>* Performing an exception return or a debug state exit to EL2.


| For EL0 or EL1 | EL1 If the controlling TBIn bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be a sign-extension of bit[55] of that address. |
| -- | -- |
| For EL2 or EL3 | If the controlling TBI bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be 0x00. |


The AddrTop() pseudocode function shows the algorithm determining the most significant bit of the VA, and
therefore whether the virtual address is using tagging. For the EL1&0 translation regime, this pseudocode includes
the selection between TTBR0_EL1 and TTBR1_EL1 described in Selection between TTBR0 and TTBR1 on
page D4-1670.

```
// AddrTop()
// =========
integer AddrTop(bits(64) address)
    // Return the MSB number of a virtual address in the current stage 1 translation
    // regime. If EL1 is using AArch64 then addresses from EL0 using AArch32
    // are zero-extended to 64 bits.
    if UsingAArch32() && !(PSTATE.EL == EL0 && !ELUsingAArch32(EL1)) then
        // AArch32 translation regime.
        return 31;
    else
        // AArch64 translation regime.
        case PSTATE.EL of
        when EL0, EL1
            tbi = if address<55> == '1' then TCR_EL1.TBI1 else TCR_EL1.TBI0;
        when EL2
            tbi = TCR_EL2.TBI;
        when EL3
            tbi = TCR_EL3.TBI;
        return (if tbi == '1' then 55 else 63);

```

>**NOTE**

>The required behavior prevents a tagged address being propagated to the program counter.


When tagging is enabled, software can use the tag bits to hold additional information about an address, provided it
ensures that any manipulation of the address correctly preserves these top bits of the address.

When address tagging is enabled for an address that causes a Data Abort or a Watchpoint, the address tag is included
in the virtual address returned in the FAR.












































































































