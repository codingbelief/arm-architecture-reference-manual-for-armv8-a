## D4.4 Access controls and memory region attributes

Translation table entry 除了包含 output address，还包含 page 或者 memory region 的属性位。这些属性位可以划分为 address map control、access control 和 region attribute 三类。 [Control of Secure or Non-secure memory access](#) 章节介绍了 address map control，后续的几个小节将对其他的属性位进行介绍：

* [Memory access control](#).
* [Memory region attributes](#).
* [Combining the stage 1 and stage 2 attributes, Non-secure EL1&0 translation regime](#).

> **NOTE:**  
本章节主要介绍各个 translation regimes 下的 access control 和 memory region attributes。一般情况下，属性的设定在 EL2 和 EL3 translation regimes 中会比较简单，在这些 translation regimes 中，还以下的一些约定：
* APTable[0] 为 RES0，并且 PE 在处理过程中会忽略该 bit, 即当该 bit 为 0 处理。
* AP[1] 为 RES1，并且 PE 在处理过程中会忽略该 bit, 即当该 bit 为 1 处理。
* PXNTable 为 RES0，并且 PE 在处理过程中会忽略该 bit, 即当该 bit 为 0 处理。
* PXN 为 RES0，并且 PE 在处理过程中会忽略该 bit, 即当该 bit 为 0 处理。