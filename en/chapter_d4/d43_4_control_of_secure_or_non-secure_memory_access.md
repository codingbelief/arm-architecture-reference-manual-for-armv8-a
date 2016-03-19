## D4.3.4 Control of Secure or Non-secure memory access

As this section describes, the NS bit in the translation table entries:
* For accesses from Secure state, if the translation table entry was held in secure memory, determines whether the access is to Secure or Non-secure memory.
* Is ignored by:
   - Accesses from Non-secure state.
   - Accesses from Secure state if the translation table entry was held in Non-secure memory.
In the VMSAv8-64 translation table format:
* The NS bit relates only to the memory block or page at the output address defined by the descriptor.
* The descriptors also include an NSTable bit, that affects accesses at lower levels of lookup, see Hierarchical control of Secure or Non-secure memory accesses on page D4-1703.
The NS and NSTable bits are valid only for memory accesses from Secure state described by translation table descriptors that are fetched from Secure memory, and:
* In the translation table descriptors in a Non-secure translation table, the NS and NSTable bits are SBZ.
* Memory accesses from Non-secure state, including all accesses from EL2, ignore the values of these bits.
In the Secure translation regimes, for translation table descriptors that are fetched from Secure memory, the NS bit in a descriptor indicates whether the descriptor refers to the Secure or the Non-secure address map, as follows:
NS == 0 Access the Secure physical address space.
NS == 1 Access the Non-secure physical address space.
For Non-secure translation regimes, and for translation table descriptors fetched from Non-secure memory, the corresponding bit is RES0 and is ignored by the PE. The access is made to Non-secure memory, regardless of the value of the bit.