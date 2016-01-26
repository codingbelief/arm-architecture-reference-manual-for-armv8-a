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

