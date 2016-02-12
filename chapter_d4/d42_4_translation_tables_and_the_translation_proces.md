## D4.2.4 Translation tables and the translation process

The following subsections describe general properties of the translation tables and translation table walks, that are largely independent of the translation table format:
* [Translation table walks](#).
* [Security state of translation table lookups on page D4-1658](#).
* [Control of translation table walks on page D4-1658](#).
* [Security state of translation table lookups on page D4-1658](#).

See also [Selection between TTBR0 and TTBR1 on page D4-1670](#).

### Translation table walks

A translation table walk comprises one or more translation table lookups. The translation table walk is the set of lookups that are required to translate the virtual address to the physical address. For the Non-secure EL1&0 translation regime, this set includes lookups for both the stage 1 translation and the stage 2 translation. The information returned by a successful translation table walk is:
* The required physical address. If the access is from Secure state this includes identifying whether the access is to the Secure physical address space or the Non-secure physical address space, see Security state of translation table lookups on page D4-1658.
* The memory attributes for the target memory region, as described in Memory types and attributes on page B2-93. For more information about how the translation table descriptors specify these attributes see Memory region attributes on page D4-1712.
* The access permissions for the target memory regions. For more information about how the translation table descriptors specify these permissions see Memory access control on page D4-1704.

