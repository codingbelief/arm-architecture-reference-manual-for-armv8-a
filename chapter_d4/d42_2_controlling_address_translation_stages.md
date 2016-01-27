> ## D4.2.2 Controlling address translation stages

## D4.2.2 Address translation stages 配置

The implemented Exception levels and the resulting translation stages and regimes on page D4-1679 defines the translation regimes and stages. For each supported address translation stage:

* A system control register bit enables the stage of address translation.
* A system control register bit determines the endianness of the translation table lookups.
* A Translation Control Register (TCR) controls the stage of address translation.
* If a stage of address translation supports splitting the VA range into two subranges then that stage of translation provides a Translation Table Base Register (TTBR) for each VA subrange, and the stage of address translation has:
   - A single TCR.
   - A TTBR for each VA subrange.
Otherwise, a single TTBR holds the address of the translation table that must be used for the first lookup for the stage of address translation.

For address translation stages controlled from AArch64:
* Table D4-1 shows the endianness bit and the enable bit for each stage of address translation. Each register entry in the table gives the endianness bit followed by the enable bit. Except for the Non-secure EL1&0 stage 2 translation, these two bits are in the same register.  
    (TODO: add table)
    > **NOTE:**  
    If the PA of the software that enables or disables a particular stage of address translation differs from its VA, speculative instruction fetching can cause complications. ARM strongly recommends that the PA and VA of any software that enables or disables a stage of address translation are identical if that stage of translation controls translations that apply to the software currently being executed.
    
| Granule size | Support indicated by: Field 
 | 2:0 | 3:0 |
| -- | -- | -- | -- |
| 0:2 | 1:2 | 2:2 | 3:2 |
| 0:3 | 1:3 | 2:3 | 3:3 |
| 0:4 | 1:4 | 2:4 | 3:4 |

* Table D4-2 shows the TCR and TTBR, or TTBRs, for each stage of address translation. In the table, each Controlling registers entry gives the TCR followed by the TTBR or TTBRs.  
    (TODO: add table)

The following subsections give more information about controlling address translation:
* System control registers relevant to MMU operation.
* Address size configuration.
* Atomicity of register changes on changing virtual machine on page D4-1650.
* Use of out-of-context translation regimes on page D4-1650.
* 


### System control registers relevant to MMU operation

In AArch64 state, system control registers have a suffix, that indicates the lowest Exception level from which they can be accessed. In some general descriptions of MMU control and address translation, this chapter uses a Common abbreviation for each of the system control registers that affects MMU operation, as Table D4-3 shows. The common abbreviation is used when describing features that apply to all the translation regimes.  

> **NOTE:**  
The only translation regime that supports a stage 2 translation is the Non-secure EL1&0 translation regime.

(TODO: add table)


### Address size configuration

The following subsubsections specify the configuration of the physical address size and of the input and output address sizes for each of the stages of address translation:
* Physical address size.
* Output address size on page D4-1647.
* Input address size on page D4-1648.
* Supported IPA size on page D4-1649.

#### Physical address size
The ID_AA64MMFR0_EL1.PARange field indicates the implemented physical address size, as Table D4-4 shows.

(TODO: add table)

All other PARange values are reserved.

#### Output address size
For each enabled stage of address translation, TCR.{I}PS must be programmed to maximum output address size for that stage of translation, using the encodings as shown in Table D4-5.

(TODO: add table)

> **NOTE:**
* This field is called IPS in the TCR_EL1, and PS in the other TCRs.
* The {I}PS fields are 3-bit fields, corresponding to the least-significant PARange bits shown in Table D4-4
on page D4-1646.

If {I}PS is programmed to a value larger than the implemented physical address size, then the PE behaves as if programmed with the implemented physical address size, but software must not rely on this behavior. That is, the output address size is never larger than the implemented physical address size. Table D4-4 on page D4-1646 shows the implemented physical address size.  
The PE checks that the TTBR, translation table entries, and the output address for the stage of address translation have the address bits above the output address size set to zero. If this is not the case, an Address size fault is generated for the level and stage of translation that caused the fault. An Address size fault from the TTBR is reported as a level 0 fault.  
If stage 1 translation is disabled and the input address is larger than the implemented physical address size, then a stage 1 level 0 Address size fault is generated.  
When using two stages of translation:
* If stage 2 translation is disabled and the output address from the stage 1 translation is larger than the implemented physical address size, then a stage 1 Address size fault is generated for the level of the stage 1 translation that generated the output address.
* If stage 2 translation is enabled and the output address from the stage 1 translation does not generate a stage 1 Address Size fault, but is larger than the input address size specified for the stage 2 translation, then a stage 2 Translation fault is generated.

#### Input address size

For each enabled stage of address translation, the TCR.TxSZ fields specify the input address size:
* TCR_EL1 has two TxSZ fields, corresponding to the two VA subranges:
    - TCR_EL1.T0SZ specifies the size for the lower VA range, translated using TTBR0_EL1.
    - TCR_EL1.T1SZ specifies the size for the upper VA range, translated using TTBR1_EL1.
* Each of the other TCRs has a single T0SZ field, and input addresses are translated using a single TTBR.
Attempting to translate an address that is larger than the configured input address size generates a Translation fault. This means:
* For a TCR with a single T0SZ field, Figure D4-3 shows the input address map:  
    (TODO: add figure)
* For a TCR with two TxSZ fields, the input address is always a VA, and Selection between TTBR0 and TTBR1 on page D4-1670 describes the VA address map.


For the Non-secure EL1&0 translation regime, when both stages of translation are enabled, if the output address from the stage 1 translation does not generate a stage 1 address size fault, and is larger than the input address specified by VTCR_EL2.T0SZ, then the input address size check for the stage 2 translation generates a Translation fault.  
Although software can configure the input address size to be smaller than 48 bits, all implemented AArch64 TTBRs must support address sizes of up to 48 bits.  
Overview of the VMSAv8-64 address translation stages on page D4-1658 gives more information about the relationship between the required input address size, the value of TxSZ, and the required initial lookup level, and how these are affected by the translation granule size. However:

**For all translation stages**  
The maximum TxSZ value is 39. If TxSZ is programmed to a value larger than 39 then it is
IMPLEMENTATION DEFINED whether:
* The implementation behaves as if the field is programmed to 39 for all purposes other than
   reading back the value of the field.
* Any use of the TxSZ value generates a Level 0 Translation fault for the stage of translation
   at which TxSZ is used.

**For a stage 1 translation**  
The minimum TxSZ value is 16. If TxSZ is programmed to a value smaller than 16 then it is
IMPLEMENTATION DEFINED whether:
* The implementation behaves as if the field were programmed to 16 for all purposes other than
reading back the value of the field.
* Any use of the TxSZ value generates a stage 1 Level 0 Translation fault.

**For a stage 2 translation**
Supported IPA size defines the effective minimum value of T0SZ, that depends on the supported PA
size, and also describes the possible effects of programming T0SZ to a value that is smaller than this
effective minimum value.

#### Supported IPA size
For the Non-secure EL1&0 translation regime, the maximum IPA size is the maximum input address size for the
second stage of translation, that must be specified by VTCR_EL2.T0SZ, see Input address size on page D4-1648.
This value is constrained by the implemented PA size that is specified by ID_AA64MMFR0_EL1.PARange, see
Physical address size on page D4-1646. This implemented PA size also constrains the maximum value of
VTCR_EL2.SL0, that specifies the level of the initial lookup. SL0 also depends on the translation granule, as
described in Overview of the VMSAv8-64 address translation stages on page D4-1658.

(TODO: add table)

If VTCR_EL2.SL0 is programmed to a value larger than the maximum value shown in Table D4-6 then any memory
access that uses the second stage of translation generates a stage 2 level 0 Translation fault.
If VTCR_EL2.T0SZ is programmed to a value smaller than the effective minimum value shown in Table D4-6 then
the implementation consistently does one of the following:
* Treat the VTCR_EL2.T0SZ field as being programmed to the effective minimum value for all purposes other
   than reading back the value of the field.
* Treat the VTCR_EL2.T0SZ field as being programmed to the effective minimum value for all purposes other
   than:
    - Reading back the value of the field.
    - Checking whether the value of VTCR_EL2.T0SZ is consistent with the value of VTCR_EL2.SL0.
* Generate a stage 2 level 0 Translation fault on any memory access that uses the second stage of translation.

> **NOTE:**  
Programming VTCR_EL2.T0SZ to a value smaller than the effective minimum value shown in Table D4-6 can
never provide support for a larger address range than the range given by the effective minimum value, because the
stage 1 output address will give an Address size fault if it is larger than either:
* The PA size, for a VMSAv8-64 stage 1 translation.
* 40 bits, for a VMSAv8-32 stage 1 translation.

#### Atomicity of register changes on changing virtual machine
From the viewpoint of software executing at Non-secure EL1 or EL0, when there is a switch from one virtual
machine to another, the registers that control or affect address translation must be changed atomically. This applies
to the registers for the Non-secure EL1&0 translation regime. This means that all of the following registers must
change atomically:
* The registers associated with the stage 1 translations:
    - MAIR_EL1 and AMAIR_EL1.
    - TTBR0_EL1, TTBR1_EL1, TCR_EL1, and CONTEXTIDR_EL1.
    - SCTLR_EL1.
* The registers associated with the stage 2 translations:
    - VTTBR_EL2 and VTCR_EL2.
    - MAIR_EL2 and AMAIR_EL2.
    - SCTLR_EL2.

> **NOTE:**  
Only some bits of SCTLR_EL1 affect the stage 1 translation, and only some bits of SCTLR_EL2 affect the stage 2
translation. However, in each case, changing these bits requires a write to the register, and that write must be atomic
with the other register updates.

These registers apply to execution using the Non-secure EL1&0 translation regime. However, when updated as part
of a switch of virtual machines they are updated by software executing at EL2. This means the registers are out of
context when they are updated, and no synchronization precautions are required.

#### Use of out-of-context translation regimes

The architecture requires that:
* When executing at EL3, EL2, or Secure EL1, the PE must not use the registers associated with the
   Non-secure EL1&0 translation regime for speculative memory accesses.
* When executing at EL3 or Secure EL1, the PE must not use the registers associated with the EL2 translation
   regime for speculative memory accesses.
* When executing at EL3, EL2, or Non-secure EL1, the PE must not use the registers associated with the
   Secure EL1 translation regime for speculative memory accesses.

When entering an Exception level, on completion of a DSB instruction, no new memory accesses using any
translation table entries from a translation regime of an Exception level lower than the Exception level that has been
entered will be observed by any observers, to the extent that those accesses are required to be observed as
determined by the shareability and cacheability of those translation table entries.

> **NOTE:**  
* This does not require that speculative memory accesses cannot be performed using those entries if it is
impossible to tell that those memory accesses have been observed by the observers.
* This requirement does not imply that, on taking an exception to a higher Exception level, any translation table
walks started before the exception was taken will be completed by the time the higher Exception level is
entered, and therefore memory accesses required for such a translation table walk might, in effect, be
performed speculatively. However, the execution of a DSB on entry to the higher Exception level ensures that
these accesses are complete.
