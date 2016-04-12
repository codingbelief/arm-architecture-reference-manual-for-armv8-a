## D4.4 Access controls and memory region attributes

In addition to an output address, a translation table entry that refers to a page or region of memory includes fields that define properties of the target memory region. These fields can be classified as address map control, access control, and region attribute fields. [Control of Secure or Non-secure memory access on page D4-1702](#) describes the address map control, and the following sections describe the other fields:  

Translation table entry 除了包含 output address，还包含 page 或者 memory region 的属性信息位。这些属性信息位可以划分为 address map control、access control 和 region attribute 三类。 [Control of Secure or Non-secure memory access](#) 章节

* [Memory access control](#).
* [Memory region attributes on page D4-1712](#).
* [Combining the stage 1 and stage 2 attributes, Non-secure EL1&0 translation regime on page D4-1717](#).

> **NOTE:**  
This section describes the access controls and memory region attributes for each of the translation regimes, and for both stages of translation in the Non-secure EL1&0 translation regime. In general, attribute assignment is simpler in the EL2 and EL3 translation regimes, and in these regimes behavior is consistent with fields in the translation tables being treated as follows:
* APTable[0] is RES0 and is ignored by the PE, meaning it is treated as if it is 0.
* AP[1] is RES1 and is ignored by the PE, meaning it is treated as if it is 1.
* The PXNTable bit is RES0 and is ignored by the PE, meaning it is treated as if it is 0.
* The PXN field is RES0 and is ignored by the PE, meaning it is treated as if it is 0.