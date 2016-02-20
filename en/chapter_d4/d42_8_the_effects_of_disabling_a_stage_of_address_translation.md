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
