## D4.2.3 Memory translation granule size

The memory translation granule size defines both:
* The maximum size of a single translation table.
* The memory page size. That is, the granularity of a translation table lookup.

VMSAv8-64 supports translation granule sizes of 4KB, 16KB, and 64KB. Support for each granule size is optional,
and is indicated as shown in Table D4-7:

![](table_d4_7.png)

In VMSAv8-64, each address translation stage is configured, independently, to use one of the supported granule
sizes.

> **NOTE:**  
* Using a larger granule size can reduce the maximum required number of levels of address lookup because:
    - The increased translation table size means the translation table holds more entries. This means a single lookup can resolve more bits of the input address.
    - The increased page size means more of the least-significant address bits are required to address a page. These address bits are flat mapped from the input address to the output address, and therefore do not require translation.
* ARM recommends that memory-mapped peripherals are separated by an integer multiple of the largest
granule size supported by the operating system or hypervisor, to allow each peripheral to be managed
independently.

Table D4-8 summarizes the effects of the different granule sizes.

![](table_d4_8.png)

### How the granule size affects the address translation process

As Table D4-8 on page D4-1651 shows, the translation granule determines the number of address bits:
* Required to address a memory page.
* That can be resolved in a single translation table lookup.

This means the translation granule determines how the input address (IA) is resolved to an output address (OA) by
the translation process.
Because a single translation table lookup can resolve only a limited number of address bits, the IA to OA resolution
requires multiple levels of lookup.
Considering the resolution of the maximum IA range of 48 bits, with a translation granule size of 2^n bytes:
* The least-significant n bits of the IA address the memory page. This means OA[(n-1):0]=IA[(n-1):0].
* The remaining (48-n) bits of the IA, IA[47:n], must be resolved by the address translation.
* A translation table descriptor is 8 bytes. Therefore:
    - A complete translation table holds 2(n-3) descriptors.
    - A single level of translation can resolve a maximum of (n-3) bits of address.  

  Consider the translation process, working back from the final level of lookup, that resolves the least
significant of the address bits that require translation. Because a level of lookup can resolve (n-3) bits of
address:
    - The final level of lookup resolves IA[(2n-4):n].
    - The previous level of lookup resolves IA[(3n-7):(2n-3)].

  However, the level of lookup that resolves the most significant bits of the IA might not require a full-sized
translation table. Therefore, in general, the address bits resolved in a level of lookup are:  
    IA[Min(47, ((x-3)(n-3)+2n-4)):(n+(x-3)(n-3))], where:
| | |
| -- | -- |
| Min(a, b)| 1:2 |
| 0:2 | 1:2 |
 Is a function that returns the minimum of a and b.
x Indicates the level of lookup. This is defined so that the level that resolves the least significant
bit of the translated IA bits is level 3.



