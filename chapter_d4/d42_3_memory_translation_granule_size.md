# D4.2.3 Memory translation granule size

The memory translation granule size defines both:
•
The maximum size of a single translation table.
•
The memory page size. That is, the granularity of a translation table lookup.
VMSAv8-64 supports translation granule sizes of 4KB, 16KB, and 64KB. Support for each granule size is optional,
and is indicated as shown in Table D4-7:
