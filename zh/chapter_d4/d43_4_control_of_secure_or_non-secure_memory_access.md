## D4.3.4 Control of Secure or Non-secure memory access

Translation table entry 中的 NS bit 有以下的用途：
* 对于 Secure state 下的内存访问，如果所属的 translation table entry 存储在 secure memory 中，那么该 bit 决定了该内存访问所访问的内存是属于 Secure 还是 Non-secure memory。
* 该 bit 在以下场景中会被忽略：
   - Non-secure state 下的内存访问.
   - 在 Secure state 的内存访问中，如果所属的 translation table entry 存储在 Non-secure memory 中.

在 VMSAv8-64 的 translation table 格式中：
* NS bit 只与 block or page table descriptor 中所定义的 block or page 的 output address 有关。
* 在 table descriptor 中，则有 NSTable bit 来描述 lower levels lookup 的属性，更多细节参考 [Hierarchical control of Secure or Non-secure memory accesses](#) 章节。

只有在 Secure state 下的内存访问，同时内存访问所使用的 translation table descriptor 存储于 Secure memory 中时，NS 和 NSTable bits 才是有效的，此外：  
* 存储在 Non-secure translation table 中的 translation table descriptor 中，NS 和 NSTable bits 都为 SBZ。
* 在 Non-secure state 下的内存访问中，包括所有来自 EL2 的内存访问，NS 和 NSTable bits 都被忽略处理。

In the Secure translation regimes, for translation table descriptors that are fetched from Secure memory, the NS bit in a descriptor indicates whether the descriptor refers to the Secure or the Non-secure address map, as follows:  
在 Secure translation regimes 中，对于存储在 Secure memory 的 translation table descriptor 中的 NS bit 指明了该 descriptor 所映射的内存是属于 Secure 还是 Non-secure，如下所示：
**NS == 0** Access the Secure physical address space.  
**NS == 1** Access the Non-secure physical address space.  

对于 Non-secure translation regimes，以及存储在 Non-secure memory 中的 translation table descriptor 中，NS 对应的 bit 为 RES0，PE 在处理过程中会忽略该 bit。访问 Non-secure memory 的内存访问不对该 bit 进行处理。

### Hierarchical control of Secure or Non-secure memory accesses

在 VMSAv8-64 的 stage 1 translation table descriptor 中，包含 NSTable bit，该 bit 用于指示该 descriptor 所指向的 table 是在 Secure 还是 Non-secure memory。在 Secure state 的内存访问中，NSTable bit 的作用如下：

**NSTable == 0**  
Descriptor 所指向的 translation table 基地址属于 Secure physical address space，该 translation table 中的 descriptors 的 NS 和 NSTable bits 为有效 bit。

**NSTable==1**  
The defined table address is in the Non-secure physical address space.Because this table is fetched from the Non-secure address space, the NS and NSTable bits in the descriptors in this table must be ignored. This means that, for this table:

* The value of the NS bit in any block or page descriptor is ignored. The block or page address refers to Non-secure memory.
* The value of the NSTable bit in any table descriptor is ignored, and the table address refers to Non-secure memory. When this table is accessed, the NS bit in any block or page descriptor is ignored, and all descriptors in the table refer to Non-secure memory.

In addition, an entry fetched in Secure state is treated as non-global if it is read from Non-secure memory. That is, these entries must be treated as if nG==1, regardless of the value of the nG bit. For more information about the nG bit, see Global and process-specific translation table entries on page D4-1730.

The effect of NSTable applies to later entries in the translation table walk, and so its effects can be held in one or more TLB entries. Therefore a change to NSTable requires coarse-grained invalidation of the TLB to ensure that the effect of the change is visible to subsequent memory transactions.