
## D4.1.1 Address tagging in AArch64 state

[Chinese](#)

In AArch64 state, the ARMv8 architecture supports tagged addresses for data values.
In these cases the top eight bits of the virtual address are ignored when determining:

 * Whether the address causes a Translation fault from being out of range if the translation system is enabled.
 * Whether the address causes an Address size fault from being out of range if the translation system is not enabled.
 * Whether the address requires invalidation when performing a TLB invalidation instruction by address.

The use of address tags is controlled as follows:

**For addresses using the VMSAv8-64 EL1&0 translation regime**

The value of bit[55] of the VA determines the register bit that controls
the use of address tags, as follows:

| | |
| -- | -- |
| VA[55]==0 | TCR_EL1.TBI0 determines whether address tags are used. If stage 1 translation is enabled, TTBR0_EL1 holds the base address of the translationtables used to translate the address.|
| VA[55]==1 | TCR_EL1.TBI1 determines whether address tags are used. If stage 1 translation is enabled, TTBR1_EL1 holds the base address of the translation tables used to translate the address.|

**For addresses using the VMSAv8-64 EL2 translation regime**

TCR_EL2.TBI determines whether address tags are used. If stage 1 translation is enabled,
TTBR0_EL2 holds the base address of the translation tables used to translate the address.


**For addresses using the VMSAv8-64 EL3 translation regime**

TCR_EL3.TBI determines whether address tags are used. If stage 1 translation is enabled,
TTBR0_EL3 holds the base address of the translation tables used to translate the address.


> **NOTE**:  
> The TCR_ELx.TBIn bits determine whether address tags are used regardless of whether the corresponding
> translation regime is enabled.

An address tag enable bit also has an effect on the PC value in the following cases:
 * Any branch or procedure return within the controlled Exception level.
 * On taking an exception to the controlled Exception level, regardless of whether this is also the Exception level from which the exception was taken.
 * On performing an exception return to the controlled Exception level, regardless of whether this is also the Exception level from which the exception return was performed.
 * Exiting from debug state to the controlled Exception level.

> **NOTE:**  

> As an example of what is meant by the controlled Exception level, TCR_EL2.TBI controls this effect for:
> * A branch or procedure return within EL2.
> * Taking an exception to EL2.
> * Performing an exception return or a debug state exit to EL2.

The effect of the controlling TBI{n} bit is:  

| | |
| -- | -- |
| For EL0 or EL1 | EL1 If the controlling TBIn bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be a sign-extension of bit[55] of that address. |
| For EL2 or EL3 | If the controlling TBI bit for the address being loaded into the PC is set to 1, then bits[63:56] of the PC are forced to be 0x00. |

The AddrTop() pseudocode function shows the algorithm determining the most significant bit of the VA, and therefore whether the virtual address is using tagging.
For the EL1&0 translation regime, this pseudocode includes the selection between TTBR0_EL1 and TTBR1_EL1 described in Selection between TTBR0 and TTBR1 on page D4-1670.

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

**Relaxation of the tagged address handling requirements on an Illegal exception return**

The AddrTop() pseudocode function, and the pseudocode description of exception return, does not cover a relaxation to the requirements for tagged address handling that applies to an Illegal exception return.
The following pseudocode describes the algorithm for the address branched to, ensuring that any address tag is not propagated to the PC:

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

> *NOTE:*
> * The TCR_ELx.TBIx fields have the effect shown in the pseudocode regardless of whether the corresponding translation regime is enabled.
> * In the case of an Illegal exception return, the tag bits of the address can be propagated to the PC if all of the following apply:

>     - The implementation treats the target_exception_level as being the Exception level that was described in the SPSR at the time of the exception return.
>     - For the Exception level that was described in the SPSR at the time of the exception return, the value
   of TCR_ELx.TBI is 0.
>     - In the Exception level that the exception was taken from, the value of TCR_ELx.TBI is 1.

>  In all other cases, the tag bits cannot be propagated to the PC.


