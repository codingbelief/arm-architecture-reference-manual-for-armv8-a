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
映射到物理内存设备的真实物理地址 (PAs) 上。这一转换过程，会因 PE 所在的
Exception levels 和所处的 Security states 的不同而不同，具体的差异可以参考 Figure D4-1。
(*TODO: 简单解释下图中的转换过程*)
![](figure_d4_1.png)

> VMSAv8-64 supports tagging of VAs,
 >as described in [Address tagging in AArch64 state](todo.md).
> As that section describes,
 > this address tagging has no effect on the address translation process.

VMSAv8-64 支持 VAs 的 tagging，address tagging 不会对地址转换的处理产生影响，后续的[D4.1.1 AArch64 下的 Address Tagging ](todo.md)小节会对此进行详细的描述。

> The remainder of this chapter gives a full description of VMSAv8-64
 > for an implementation that includes all of the Exception levels.
> [The implemented Exception levels and the resulting translation stages and regimes on
 >page D4-1679](todo.md) describes the differences in the VMSA
 > if some Exception levels are not implemented.
 
本章节后续的内容将对实现了所有 Exception levels 的系统上的 VMSAv8-64 进行详细的介绍。
对于只实现了部分 Exception levels 的系统上的  VMSAv8-64，可以参考
[The implemented Exception levels and the resulting translation stages and regimes on page D4-1679](todo.md) 
章节的描述

> # D4.1.1 Address tagging in AArch64 state
# D4.1.1 AArch64 下的地址 tagging

> In AArch64 state, the ARMv8 architecture supports tagged addresses for data 
 > values. In these cases the top eight
> bits of the virtual address are ignored when determining:
> * Whether the address causes a Translation fault from being out of range if the translation system is enabled.
> * Whether the address causes an Address size fault from being out of range if the translation system is not enabled.
> * Whether the address requires invalidation when performing a TLB invalidation instruction by address.

在 AArch64 运行态下，ARMv8 架构支持虚拟地址的 tagging 操作。在启用了 VA 的 tagging 时，VA 的高8个bits将作为 tag 使用，同时将不参与下列的处理过程：
* 当地址转换使能时，判定 VA 超出范围后产生 Translation fault。
* 当地址转换未使能时，判定 VA 超出范围后产生 Address size fault。
* 当进行 TLB 设置指定地址的条目失效操作时。

(译者注：VA 的 tag 通常是在软件层面使用，例如用来作为对象的引用计数、标示指针的有效与否等)

> The use of address tags is controlled as follows:

> **For addresses using the VMSAv8-64 EL1&0 translation regime**

> The value of bit[55] of the VA determines the register bit that controls the use of address tags, as
> follows:

> | VA[55]==0 | TCR_EL1.TBI0 determines whether address tags are used. If stage 1 translation is enabled, TTBR0_EL1 holds the base address of the translationtables used to translate the address.|
> | -- | -- |
> | VA[55]==1 | TCR_EL1.TBI1 determines whether address tags are used. If stage 1 translation is enabled, TTBR1_EL1 holds the base address of the translation tables used to translate the address.|

虚拟地址的 tag 主要通过以下的方式来配置：

**对于使用 VMSAv8-64 EL1&0 translation regime 的地址**  
虚拟地址的 tag 功能的配置会根据 bit[55] 的值的不同而不同：

 | VA[55]==0 |如果使能了 stage 1 的转换, 那么 TCR_EL1.TBI0 决定是否启用 address tags 功能. 寄存器 TTBR0_EL1 保存地址转换表的基地址.|
 | -- | -- |
 | VA[55]==1 |如果使能了 stage 1 的转换, 那么 TCR_EL1.TBI1 决定是否启用 address tags 功能. 寄存器 TTBR1_EL1 保存地址转换表的基地址.|
 

> **For addresses using the VMSAv8-64 EL2 translation regime**

> TCR_EL2.TBI determines whether address tags are used. If stage 1 translation is enabled,
> TTBR0_EL2 holds the base address of the translation tables used to translate the address.


> **For addresses using the VMSAv8-64 EL3 translation regime**

> TCR_EL3.TBI determines whether address tags are used. If stage 1 translation is enabled,
> TTBR0_EL3 holds the base address of the translation tables used to translate the address.


>> **NOTE**:

>> The TCR_ELx.TBIn bits determine whether address tags are used regardless of whether the corresponding
>> translation regime is enabled.


**对于使用 VMSAv8-64 EL2 translation regime 的地址**  
如果使能了 stage 1 的转换，那么 TCR_EL2.TBI 决定了是否启用 address tags 功能，
寄存器 TTBR0_EL2 保存地址转换表的基地址。

**对于使用 VMSAv8-64 EL3 translation regime 的地址**  
如果使能了 stage 1 的转换，那么 TCR_EL3.TBI 决定了是否启用 address tags 功能，
寄存器 TTBR0_EL3 保存地址转换表的基地址。
> **NOTE**:

> 不管对应的 translation regime 是否使能，寄存器位 TCR_ELx.TBIn 都决定着 address tags 功能是否启用.
  
--
> An address tag enable bit also has an effect on the PC value in the following cases:
> * Any branch or procedure return within the controlled Exception level.
> * On taking an exception to the controlled Exception level, regardless of whether this is also the Exception level from which the exception was taken.
> * On performing an exception return to the controlled Exception level, regardless of whether this is also the Exception level from which the exception return was performed.
> * Exiting from debug state to the controlled Exception level.

在下面的场景中，address tags 功能的开启与否，也会影响 PC 寄存器的值：
* Any branch or procedure return within the controlled Exception level.
* On taking an exception to the controlled Exception level, regardless of whether this is also the Exception level from which the exception was taken.
* On performing an exception return to the controlled Exception level, regardless of whether this is also the Exception level from which the exception return was performed.
* Exiting from debug state to the controlled Exception level.  
（TODO: 此处的 controlled Exception level 不理解，待后续进行翻译）

>> **NOTE:**

>> As an example of what is meant by the controlled Exception level, TCR_EL2.TBI controls this effect for:
>> * A branch or procedure return within EL2.
>> * Taking an exception to EL2.
>> * Performing an exception return or a debug state exit to EL2.


>>  | For EL0 or EL1 | EL1 If the controlling TBIn bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be a sign-extension of bit[55] of that address. |
>>  | -- | -- |
>>  | For EL2 or EL3 | If the controlling TBI bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be 0x00. |


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


**Relaxation of the tagged address handling requirements on an Illegal exception return**

The AddrTop() pseudocode function, and the pseudocode description of exception return, does not cover a relaxation
to the requirements for tagged address handling that applies to an Illegal exception return. The following
pseudocode describes the algorithm for the address branched to, ensuring that any address tag is not propagated to
the PC:

```
if (target_exception_level == EL0) || (target_exception_level == EL1) then
    if NewAddress<55> == '1' && TCR_EL1.TBI1== '1' then
        NewAddress<63:56> = '11111111';
    if NewAddress<55> == '0' && TCR_EL1.TBI0== '1' then
        NewAddress<63:56> = '00000000';
if (target_exception_level == EL2) then
    if TCR_EL2.TBI== '1' then
        NewAddress<63:56> = '00000000';
if (target_exception_level == EL3) then
    if TCR_EL3.TBI== '1' then
        NewAddress<63:56> = '00000000';
PC = NewAddress;

```

In this pseudocode:

*NewAddress*:

Is the address being branched to, or returned to.

*target_exception_level*:
* Is the current Exception level for a branch or procedure return.
* Is the Exception level being returned to for an exception return.
   If the exception return triggers the Illegal exception return mechanism, it is IMPLEMENTATION
  DEFINED whether target_exception_level is the Exception level that was described in the
 SPSR at the time of the exception return or the Exception level that the exception return
instruction was executed from.
* Is the Exception level the exception is taken to for an exception entry

>NOTE:
>* The TCR_ELx.TBIx fields have the effect shown in the pseudocode regardless of whether the corresponding translation regime is enabled.
>* In the case of an Illegal exception return, the tag bits of the address can be propagated to the PC if all of the following apply:

>     - The implementation treats the target_exception_level as being the Exception level that was described
   in the SPSR at the time of the exception return.
>     - For the Exception level that was described in the SPSR at the time of the exception return, the value
   of TCR_ELx.TBI is 0.
>     - In the Exception level that the exception was taken from, the value of TCR_ELx.TBI is 1.

>  In all other cases, the tag bits cannot be propagated to the PC.










































































































