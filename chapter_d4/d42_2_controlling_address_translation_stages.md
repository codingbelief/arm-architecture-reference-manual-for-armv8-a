## D4.2.2 Controlling address translation stages

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
    

* Table D4-2 shows the TCR and TTBR, or TTBRs, for each stage of address translation. In the table, each Controlling registers entry gives the TCR followed by the TTBR or TTBRs.

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
