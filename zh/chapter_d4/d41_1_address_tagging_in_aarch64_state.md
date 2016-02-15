
> ## D4.1.1 Address tagging in AArch64 state

## D4.1.1 AArch64 下的地址 tagging

[`英文版`](../../en/chapter_d4/d41_1_address_tagging_in_aarch64_state.html)

> In AArch64 state, the ARMv8 architecture supports tagged addresses for data 
 > values. In these cases the top eight
> bits of the virtual address are ignored when determining:
> * Whether the address causes a Translation fault from being out of range if the translation system is enabled.
> * Whether the address causes an Address size fault from being out of range if the translation system is not enabled.
> * Whether the address requires invalidation when performing a TLB invalidation instruction by address.

在 AArch64 运行态下，ARMv8 架构支持 Address Tagging 功能。在启用了 VA 的 tagging 时，VA 的高 8 个 bits 将作为 tag 使用，同时在下列的几个处理过程，高 8 bits 将被忽略：
* 当地址转换使能时，判定 VA 是否超出范围，并产生 Translation fault。
* 当地址转换未使能时，判定 VA 是否超出范围，并产生 Address size fault。
* 当进行 TLB 设置指定地址的条目失效操作时。

(译者注：VA 的 tag 通常是在软件层面使用，例如用来作为对象的引用计数、标示指针的有效与否等)

> The use of address tags is controlled as follows:

> **For addresses using the VMSAv8-64 EL1&0 translation regime**

> The value of bit[55] of the VA determines the register bit that controls the use of address tags, as
> follows:

> | | |
> | -- | -- |
> | VA[55]==0 | TCR_EL1.TBI0 determines whether address tags are used. If stage 1 translation is enabled, TTBR0_EL1 holds the base address of the translationtables used to translate the address.|
> | VA[55]==1 | TCR_EL1.TBI1 determines whether address tags are used. If stage 1 translation is enabled, TTBR1_EL1 holds the base address of the translation tables used to translate the address.|

Address tagging 功能通过以下的方式来配置：

**对于使用 VMSAv8-64 EL1&0 translation regime 的地址**  

在 EL1&0 下，VMSAv8-64 支持两个区块的虚拟地址空间，分别为：VA 高 16 位全为 0 的区块和 VA 高 16 位全为 1 的区块， 不同区块的的 address tagging 功能的配置由不同的寄存器负责：

|  |  |
| -- | -- |
| VA[55]==0，即高 16 位全为 0 的区块 | 如果使能了 stage 1 的转换, 那么 TCR_EL1.TBI0 控制是否启用 address tags 功能. 寄存器 TTBR0_EL1 保存地址转换表的基地址.|
| VA[55]==1，即高 16 位全为 1 的区块 | 如果使能了 stage 1 的转换, 那么 TCR_EL1.TBI1 控制是否启用 address tags 功能. 寄存器 TTBR1_EL1 保存地址转换表的基地址.|
 
（译者注：TCR for Translation Control Register，TBI for Top Byte Ignore，TTBR for Translation Table Base Register）
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

如果使能了 stage 1 的转换，那么 TCR_EL2.TBI 控制是否启用 address tags 功能，
寄存器 TTBR0_EL2 保存地址转换表的基地址。

**对于使用 VMSAv8-64 EL3 translation regime 的地址**

如果使能了 stage 1 的转换，那么 TCR_EL3.TBI 控制是否启用 address tags 功能，
寄存器 TTBR0_EL3 保存地址转换表的基地址。
> **NOTE**:  

> 不管对应的 translation regime 是否使能，寄存器位 TCR_ELx.TBIn 都决定着 address tags 功能是否启用.
  
--
> An address tag enable bit also has an effect on the PC value in the following cases:
> * Any branch or procedure return within the controlled Exception level.
> * On taking an exception to the controlled Exception level, regardless of whether this is also the Exception level from which the exception was taken.
> * On performing an exception return to the controlled Exception level, regardless of whether this is also the Exception level from which the exception return was performed.
> * Exiting from debug state to the controlled Exception level.

开启 address tagging 功能后，由于 VA 的高 8 位被用作为 tag，因此在下列需要更新 PC 寄存器的场景中，需要对写入 PC 的值做特殊的处理：
* 开启了 address tagging 功能的 Exception level 下的所有的 branch 和 procedure return 操作。
* 通过 exception 陷入到开启了 address tagging 功能的 Exception level 时。
* exception 返回到开启了 address tagging 功能的 Exception level 时.
* 从 debug 状态退出，并返回到开启了 address tagging 功能的 Exception level 时。

>> **NOTE:**  

>> As an example of what is meant by the controlled Exception level, TCR_EL2.TBI controls this effect for:
>> * A branch or procedure return within EL2.
>> * Taking an exception to EL2.
>> * Performing an exception return or a debug state exit to EL2.

> The effect of the controlling TBI{n} bit is:  

> | | |
> | -- | -- |
> | For EL0 or EL1 | EL1 If the controlling TBIn bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be a sign-extension of bit[55] of that address. |
> | For EL2 or EL3 | If the controlling TBI bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be 0x00. |

> **NOTE:**  

> 下面以 Exception level 2 为例, 在设置 TCR_EL2.TBI 使能了 address tagging 功能后，在下面的场景中，需要对写入 PC 寄存器的值做特殊的处理:
> * EL2 中发生的 branch 和 procedure return.
> * 发生 exception 陷入到 EL2.
> * 从 exception 或者 debug 状态退出，并返回到 EL2.

在不同的 EL 中使能 address tagging 功能后，在上述场景中更新 PC 寄存器的值时，需要的对写入 PC 寄存器的值所做的特殊处理如下表所示：

| | |
| -- | -- |
| For EL0 or EL1 |如果 address tagging 使能，即 TBI 被设置为 1，那么写入 PC 的地址值得高 8 位将设置为与 bit[55] 一样|
| For EL2 or EL3 |如果 address tagging 使能，即 TBI 被设置为 1，那么写入 PC 的地址值得高 8 位将设置为 0 |

> The AddrTop() pseudocode function shows the algorithm determining the most significant bit of the VA, and therefore whether the virtual address is using tagging.
> For the EL1&0 translation regime, this pseudocode includes the selection between TTBR0_EL1 and TTBR1_EL1 described in Selection between TTBR0 and TTBR1 on page D4-1670.

下面的伪代码算法描述了 address tagging 的开启与否，是如何影响 VA 的 MSB 的计算的。在 EL1&0 translation regime 中，下面的算法适用于从 TTBR0_EL1 寄存器所设定的地址开始，到 TTBR1_EL1 寄存器所设定的地址结束的所有 VA。
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


When tagging is enabled, software can use the tag bits to hold additional information about an address, provided it ensures that any manipulation of the address correctly preserves these top bits of the address.

When address tagging is enabled for an address that causes a Data Abort or a Watchpoint, the address tag is included in the virtual address returned in the FAR.

Address tagging 使能后，软件可以使用 VA 的高8位来保存额外的信息，但是软件需要保证在直接操作保存有 tag 信息的 VA 时，VA 的高8位没有被错误的更改。另外，当发生 Data Abort 和 Watchpoint 时，tag 信息会和 VA 一起被保存到 FAR ( Fault Address Register )。

>**Relaxation of the tagged address handling requirements on an Illegal exception return**

> The AddrTop() pseudocode function, and the pseudocode description of exception return, does not cover a relaxation to the requirements for tagged address handling that applies to an Illegal exception return.
>The following pseudocode describes the algorithm for the address branched to, ensuring that any address tag is not propagated to the PC:

**Relaxation of the tagged address handling requirements on an Illegal exception return**

The AddrTop() pseudocode function, and the pseudocode description of exception return, does not cover a relaxation to the requirements for tagged address handling that applies to an Illegal exception return.
(TODO：此处不是很明白所表达的意思，是想说明 Illegal exception return 中对 tagged address 的处理有不同的方式么？)

下面的伪代码算法描述了当跳转到带有 tag 信息的 VA 执行时，是如何保证 VA 的 tag 信息不会被加载到 PC 寄存器中的：

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

> In this pseudocode:

> *NewAddress*:

> Is the address being branched to, or returned to.

> *target_exception_level*:
> * Is the current Exception level for a branch or procedure return.
> * Is the Exception level being returned to for an exception return.  
    If the exception return triggers the Illegal exception return mechanism, it is IMPLEMENTATION
    DEFINED whether target_exception_level is the Exception level that was described in the
    SPSR at the time of the exception return or the Exception level that the exception return
instruction was executed from.
> * Is the Exception level the exception is taken to for an exception entry

在上面的伪代码中：

*NewAddress*：
指的是跳转到的目的 VA，或者调用返回时的目的 VA。

*target_exception_level* 则可能有以下几种情况：
* branch 或者 procedure return 所在的 Exception level
* exception return 所要返回的 Exception level。  
  如果在 exception return 时触发了 Illegal exception return 机制，那么 target_exception_level 可能是 exception return 时 SPSR 寄存器所指示的 Exception level，也可能是 exception return 执行时所在的 Exception level，选择哪一个由具体实现决定。
* exception entry 所要进入的 Exception level。


>> *NOTE:*
>> * The TCR_ELx.TBIx fields have the effect shown in the pseudocode regardless of whether the corresponding translation regime is enabled.
>> * In the case of an Illegal exception return, the tag bits of the address can be propagated to the PC if all of the following apply:

>>     - The implementation treats the target_exception_level as being the Exception level that was described in the SPSR at the time of the exception return.
>>     - For the Exception level that was described in the SPSR at the time of the exception return, the value
   of TCR_ELx.TBI is 0.
>>     - In the Exception level that the exception was taken from, the value of TCR_ELx.TBI is 1.

>>  In all other cases, the tag bits cannot be propagated to the PC.

> *NOTE:*
> * 不管 translation regime 是否使能，寄存器 TCR_ELx 的 TBIx 位都会参与上面伪代码的算法处理过程。
> * 在 Illegal exception return 中, 如果下面的几个条件都符合，那么 VA 的 tag bits 会被加载到 PC 寄存器中:

>     - 在具体的实现中，exception return 时的 target_exception_level 被实现为 SPSR 寄存器所指示的 Exception level
>     - exception return 时的 SPSR 寄存器所指示的 Exception level 中的 TCR_ELx.TBI 被设定为 0。
>     - 产生该 exception 的 Exception level 中的 TCR_ELx.TBI 被设定为 1.  
>     (译者注：即从一个使能了address tagging 的 EL 发生 exception 陷入的到另一个没有使能 address tagging 的 EL 时，在该 exception 返回时，会出现 带有 tag bits 的 VA 从 ELR 寄存器加载到 PC 寄存器的情况。)  
>  在其他的场景下，VA 的 tag bits 都不会被加载到 PC 寄存器中。


