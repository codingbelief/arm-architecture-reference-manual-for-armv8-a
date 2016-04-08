## D4.3.4 Control of Secure or Non-secure memory access

As this section describes, the NS bit in the translation table entries:
* For accesses from Secure state, if the translation table entry was held in secure memory, determines whether the access is to Secure or Non-secure memory.
* Is ignored by:
   - Accesses from Non-secure state.
   - Accesses from Secure state if the translation table entry was held in Non-secure memory.

Translation table entry 中的 NS bit 由以下的用途：
* 

In the VMSAv8-64 translation table format:
* The NS bit relates only to the memory block or page at the output address defined by the descriptor.
* The descriptors also include an NSTable bit, that affects accesses at lower levels of lookup, see [Hierarchical control of Secure or Non-secure memory accesses on page D4-1703](#).

The NS and NSTable bits are valid only for memory accesses from Secure state described by translation table descriptors that are fetched from Secure memory, and:
* In the translation table descriptors in a Non-secure translation table, the NS and NSTable bits are SBZ.
* Memory accesses from Non-secure state, including all accesses from EL2, ignore the values of these bits.

In the Secure translation regimes, for translation table descriptors that are fetched from Secure memory, the NS bit in a descriptor indicates whether the descriptor refers to the Secure or the Non-secure address map, as follows:  
**NS == 0** Access the Secure physical address space.  
**NS == 1** Access the Non-secure physical address space.  

For Non-secure translation regimes, and for translation table descriptors fetched from Non-secure memory, the corresponding bit is RES0 and is ignored by the PE. The access is made to Non-secure memory, regardless of the value of the bit.

### Hierarchical control of Secure or Non-secure memory accesses

For VMSAv8-64 table descriptors for stage 1 translations, the descriptor includes an NSTable bit, that indicates whether the table identified in the descriptor is in Secure or Non-secure memory. For accesses from Secure state, the meaning of the NSTable bit is:

**NSTable == 0**  
The defined table address is in the Secure physical address space. In the descriptors in that translation table, NS bits and NSTable bits have their defined meanings.

**NSTable==1**  
ThedefinedtableaddressisintheNon-securephysicaladdressspace.Becausethistableisfetched from the Non-secure address space, the NS and NSTable bits in the descriptors in this table must be ignored. This means that, for this table:
* The value of the NS bit in any block or page descriptor is ignored. The block or page address refers to Non-secure memory.
* The value of the NSTable bit in any table descriptor is ignored, and the table address refers to Non-secure memory. When this table is accessed, the NS bit in any block or page descriptor is ignored, and all descriptors in the table refer to Non-secure memory.

In addition, an entry fetched in Secure state is treated as non-global if it is read from Non-secure memory. That is, these entries must be treated as if nG==1, regardless of the value of the nG bit. For more information about the nG bit, see Global and process-specific translation table entries on page D4-1730.

The effect of NSTable applies to later entries in the translation table walk, and so its effects can be held in one or more TLB entries. Therefore a change to NSTable requires coarse-grained invalidation of the TLB to ensure that the effect of the change is visible to subsequent memory transactions.