# D4.2.3 Memory translation granule size

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

