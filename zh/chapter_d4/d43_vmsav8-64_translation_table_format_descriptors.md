# D4.3 VMSAv8-64 translation table format descriptors

In general, a descriptor is one of:
通常情况下，一个 descriptor 可能是以下的几种 entry：
* 非法或者异常的 entry。
* Table entry, 指向 next-level translation table。
* Block entry, 定义内存访问的 memory properties。
* Reserved format。

Descriptor 的 bit[1] 用于指示 descriptor 的类型，bit[0] 用于指示 descriptor 是否有效。

下面的几个小节主要描述了 ARMv8 translation table descriptor 的格式：

* [VMSAv8-64 translation table level 0, level 1, and level 2 descriptor formats](#).
* [ARMv8 translation table level 3 descriptor formats](#).

[Memory attribute fields in the VMSAv8-64 translation table format descriptors](#) 章节中描述了 descriptor attribute fields 的更多细节，[Control of Secure or Non-secure memory access](#) 章节则 NS 和 NSTable 是怎样控制在 Secure state 下的对 Secure 和 Non-secure memory 的访问。