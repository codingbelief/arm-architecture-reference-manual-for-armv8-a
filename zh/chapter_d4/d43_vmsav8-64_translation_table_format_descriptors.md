# D4.3 VMSAv8-64 translation table format descriptors

In general, a descriptor is one of:
通常情况下，一个 descriptor 可能是以下的几种 entry：
* 非法或者异常的 entry。
* Table entry, 指向 next-level translation table。
* Block entry, 定义内存访问的 memory properties。
* Reserved format。

Bit[1] of the descriptor indicates the descriptor type, and bit[0] indicates whether the descriptor is valid.

The following sections describe the ARMv8 translation table descriptor formats:

* [VMSAv8-64 translation table level 0, level 1, and level 2 descriptor formats](#).
* [ARMv8 translation table level 3 descriptor formats on page D4-1698](#).

[Memory attribute fields in the VMSAv8-64 translation table format descriptors on page D4-1699](#) then gives more information about the descriptor attribute fields, and [Control of Secure or Non-secure memory access on page D4-1702](#) describe how the NS and NSTable together control whether a memory access from Secure state accesses the Secure memory map or the Non-secure memory map.
