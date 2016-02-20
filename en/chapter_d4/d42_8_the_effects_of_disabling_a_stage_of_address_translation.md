## D4.2.8 The effects of disabling a stage of address translation

The following sections describe the effect on MMU behavior of disabling each stage of translation:
* [Behavior when stage 1 address translation is disabled](#).
* [Behavior when stage 2 address translation is disabled on page D4-1678](#).
* [Behavior of instruction fetches when all associated stages of translation are disabled on page D4-1679](#).

### Behavior when stage 1 address translation is disabled

When a stage 1 address translation is disabled, memory accesses that would otherwise be translated by that stage of translation are treated as follows:


#### Non-secure EL1 and EL0 accesses if the HCR_EL2.DC bit is set to 1

For the Non-secure EL1&0 translation regime, when the value of HCR_EL2.DC is 1, the stage 1 translation assigns the Normal Non-shareable, Inner Write-Back Read-Write-Allocate, Outer Write-Back Read-Write-Allocate memory attributes.

> **NOTE: **  
This applies for both instruction and data accesses.


#### All other accesses

For all other accesses, when stage 1 address translation is disabled, the assigned attributes depend on whether the access is a data access or an instruction access, as follows:

**Data access**
The stage 1 translation assigns the Device-nGnRnE memory type.

**Instruction access**
The stage 1 translation assigns the Normal memory attribute, with the cacheability and shareability attributes determined by the value of the SCTLR.I bit for the translation regime, as follows:
**When the value of I is 0**: The stage 1 translation assigns the Non-cacheable and Outer Shareable attributes.

**When the value of I is 1**: The stage 1 translation assigns the Cacheable, Inner Write-Through no Write-Allocate Read-Allocate, Outer Write-Through no Write-Allocate Read Allocate Outer Shareable attribute.

For this stage of translation, no memory access permission checks are performed, and therefore no MMU faults can be generated for this stage of address translation.

> **NOTE: **  
Alignment checking is performed, and therefore Alignment faults can occur.

For every access, the input address of the stage 1 translation is flat-mapped to the output address.

For a Non-secure EL1 or EL0 access, if EL1&0 stage 2 address translation is enabled, the stage 1 memory attribute assignments and output address can be modified by the stage 2 translation.

When the value of HCR_EL2.DC is 1, in Non-secure state:
* The SCTLR_EL1.M bit behaves as if it is 0, for all purposes other than reading the value of the bit. This means Non-secure EL1&0 stage 1 address translation is disabled.
* The HCR_EL2.VM bit behaves as if it is 1, for all purposes other than reading the value of the bit. This means that Non-secure EL1&0 stage 2 address translation is enabled.

See also [Behavior of instruction fetches when all associated stages of translation are disabled on page D4-1679](#).

***Effect of disabling address translation on maintenance and address translation instruction
instructions***

Cache maintenance instructions act on the target cache regardless of whether any stages of address translation are disabled, and regardless of the values of the memory attributes. However, if a stage of address translation is disabled, they use the flat address mapping for that translation stage.
TLB invalidate operations act on the target TLB regardless of whether any stage of address translation is disabled. The value of HCR_EL2.DC affect some address translation instructions, see [Address translation instructions, AT* on page D4-1692](#).


### Behavior when stage 2 address translation is disabled

When stage 2 address translation is disabled:
* The IPA output from the stage 1 translation maps flat to the PA.
* The memory attributes and permissions from the stage 1 translation apply to the PA.
When both stages of address translation are disabled, see also [Behavior of instruction fetches when all associated stages of translation are disabled on page D4-1679](#).


### Behavior of instruction fetches when all associated stages of translation are disabled

When EL3 is using AArch64, this section applies to:
* The Secure EL1&0 translation regime when Secure EL1&0 stage 1 address translation is disabled.
* The Secure EL3 translation regime, when Secure EL3 stage 1 address translation is disabled.
* The Non-secure EL2 translation regime, when Non-secure EL2 stage 1 address translation is disabled
* The Non-secure EL1&0 translation regime, when both stages of address translation are disabled.

> **NOTE: **  
* The behaviors in Non-secure state apply regardless of the Execution state that EL3 is using.
* When the value of HCR_EL2.DC is 1, then the behavior of the Non-secure EL1&0 translation regime is as if stage 1 translation is disabled and stage 2 translation is enabled, as described in [Behavior when stage 1 address translation is disabled on page D4-1677](#).





