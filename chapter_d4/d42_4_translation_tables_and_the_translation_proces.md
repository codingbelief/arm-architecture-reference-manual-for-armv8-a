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

The translation table walk starts with a read of the translation table for the initial lookup. The TTBR for the stage of translation holds the base address of this table. Each translation table lookup returns a descriptor, that indicates one of the following:

• The entry is the final entry of the walk. In this case, the entry contains the OA, and the permissions and attributes for the access.
• An additional level of lookup is required. In this case, the entry contains the translation table base address for that lookup. In addition:
— The descriptor provides hierarchical attributes that are applied to the final translation, see Hierarchical control of Secure or Non-secure memory accesses on page D4-1703 and Hierarchical control of data access permissions on page D4-1706.
— If the translation is in a Secure translation regime, the descriptor indicates whether that base address is in the Secure or Non-secure address space, unless a hierarchical control at a previous level of lookup has indicated that it must be in the Non-secure address space.
• The descriptor is invalid. In this case, the memory access generates a Translation fault.
Figure D4-7 on page D4-1657 gives a generalized view of a single stage of address translation, where three levels of lookup are required.
