## D4.2.6 The VMSAv8-64 translation table format

This section provides the full description of the VMSAv8-64 translation table format, its use for address translations that are controlled by an Exception level using AArch64.
For the address translations that are controlled by an Exception level that is using AArch64:
* The TCR_EL1.{SH0, ORGN0, IRGN0, SH1, ORGN1, IRGN1} fields define memory region attributes for
the translation table walk, for each of TTBR0_EL1 and TTBR1_EL1.
* For the Secure and Non-secure EL1&0 stage 1 translations, each of TTBR0_EL1 and TTBR1_EL1 contains
an ASID field, and the TCR_EL1.A1 field selects which ASID to use.

For this translation table format, Overview of the VMSAv8-64 address translation stages on page D4-1658 summarizes the lookup levels, and Descriptor encodings, ARMv8 level 0, level 1, and level 2 formats on page D4-1696 describes the translation table entries.
The following subsections describe the use of this translation table format:

* Translation granule size and associate block and page sizes on page D4-1668.
* Selection between TTBR0 and TTBR1 on page D4-1670.
* Concatenated translation tables for the initial stage 2 lookup on page D4-1671.
* Possible translation table registers programming errors on page D4-1673.

### Translation granule size and associate block and page sizes

Table D4-20 shows the supported granule sizes, block sizes and page sizes, for the different granule sizes. For completeness, this table includes information for AArch32 state. In the table, the OA bit ranges are the OA bits that the translation table descriptor specifies to address the block or page of memory, in an implementation that supports a 48-bit OA range.

![](table_d4_20.png)

Bit[1] of a translation table descriptor identifies whether the descriptor is a block descriptor, and:
* The 4KB granule size supports block descriptors only in level 1and level 2 translation tables.
* The 16KB and 64KB granule sizes support block descriptors only in level2 translation tables,
Setting bit[1] of a descriptor to 0 in a translation table that does not support block descriptors gives a Translation fault.
For translations managed from AArch64 state, the following tables expand the information for each granule size, showing for each lookup level and when accessing a single translation table:
* The maximum IA size, and the address bits that are resolved for that maximum size.
* The maximum OA range resolved by the translation table descriptors at this level, and the corresponding
memory region size.
* The maximum size of the translation table. This is the size required for the maximum IA size.
Table D4-21 shows this information for the 4KB translation granule size, Table D4-22 on page D4-1669 shows this information for the 16KB translation granule size, and Table D4-23 on page D4-1669 shows this information for the 64KB translation granule size.

![](table_d4_21_1.png)
![](table_d4_21_2.png)

![](table_d4_22.png)
![](table_d4_23.png)

For the initial lookup level:
* If the IA range specified by the TCR.TxSZ field is smaller than the maximum size shown in these table then this reduces the number of addresses in the table and therefore reduces the table size. The smaller translation table is aligned to its table size.
* For stage 2 translations, multiple translation tables can be concatenated to extend the maximum IA size beyond that shown in these tables. For more information see the stage 2 translation overviews in Overview of the VMSAv8-64 address translation stages on page D4-1658 and Concatenated translation tables for the initial stage 2 lookup on page D4-1671.

If a supplied input address is larger than the configured input address size, a Translation fault is generated.

