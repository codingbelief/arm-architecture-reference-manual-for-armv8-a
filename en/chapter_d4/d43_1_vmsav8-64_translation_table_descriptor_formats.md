# D4.3.1 VMSAv8-64 translation table level 0 level 1 and level 2 descriptor formats

In the VMSAv8-64 translation table format, the difference in the formats of the level 0, level 1 and level 2 descriptors is:
* Whether a block entry is permitted.
* If a block entry is permitted, the size of the memory region described by that entry.

These differences depend on the translation granule, as follows:  
**4KB granule** A level 0 descriptor does not support block translation.
A block entry:
* In a level 1 table describes the mapping of the associated 1GB input address range.
* In a level 2 table describes the mapping of the associated 2MB input address range.  

**16KB granule** Level 0 and level 1 descriptors do not support block translation.
A block entry in a level 2 table describes the mapping of the associated 32MB input address range.  
**64KB granule** Level 0 lookup is not supported.
A level 1 descriptor does not support block translation.
A block entry in a level 2 table describes the mapping of the associated 512MB input address range.
