# D4.3.1 VMSAv8-64 translation table level 0 level 1 and level 2 descriptor formats

In the VMSAv8-64 translation table format, the difference in the formats of the level 0, level 1 and level 2 descriptors is:
* Whether a block entry is permitted.
* If a block entry is permitted, the size of the memory region described by that entry.

These differences depend on the translation granule, as follows:

