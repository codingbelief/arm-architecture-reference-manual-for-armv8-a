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
Descriptor 所指向的 translation table 基地址属于 Non-secure physical address space。由于该 translation table 存储在 Non-secure memory，因此该 table 中的所有 descriptors 的 NS 和 NSTable bits 都会被忽略。这也就意味着：
* 该 table 中的所有 block or page descriptor 的 NS bit 都会被忽略。Block or page address 都属于 Non-secure memory。 
* 该 table 中的 table descriptor 的 NSTable 都会被忽略，table descriptor 所指向的 table address 属于 Non-secure memory。当 table descriptor 所指向的 table 被访问时，此 table 的 block or page descriptor 的 NS bit 也会被忽略，此 table 的所有 descriptors 所映射的地址都属于 Non-secure memory。

此外，在 Secure state 下，从 Non-secure memory 中读取的 translation table entry 都会当做 non-global 处理，即不管该 entry 中的 nG bit 的值是多少，都当 nG 为 1 来处理。更多 nG bit 相关的信息，可以参考 [Global and process-specific translation table entries](#) 章节。

The effect of NSTable applies to later entries in the translation table walk, and so its effects can be held in one or more TLB entries. Therefore a change to NSTable requires coarse-grained invalidation of the TLB to ensure that the effect of the change is visible to subsequent memory transactions.

由于 NSTable 的状态会影响后续 translation table walk 中的 entries，包括已经被加载到 TLB 中 entries。因此，在修改 NSTable bit 时，需要做 coarse-grained invalidation of the TLB 操作