## D4.3.1 VMSAv8-64 translation table level 0 level 1 and level 2 descriptor formats

In the VMSAv8-64 translation table format, the difference in the formats of the level 0, level 1 and level 2 descriptors is:
* Whether a block entry is permitted.
* If a block entry is permitted, the size of the memory region described by that entry.

These differences depend on the translation granule, as follows:  
**4KB granule**  
A level 0 descriptor does not support block translation.  
A block entry:
* In a level 1 table describes the mapping of the associated 1GB input address range.
* In a level 2 table describes the mapping of the associated 2MB input address range.  

**16KB granule**  
Level 0 and level 1 descriptors do not support block translation.  
A block entry in a level 2 table describes the mapping of the associated 32MB input address range.  

**64KB granule**  
Level 0 lookup is not supported.  
A level 1 descriptor does not support block translation.  
A block entry in a level 2 table describes the mapping of the associated 512MB input address range.

[Figure D4-16 on page D4-1696](#) shows the ARMv8 level 0, level 1, and level 2 descriptor formats:

![](figure_d4_16.png)

### Descriptor encodings, ARMv8 level 0, level 1, and level 2 formats

Descriptor bit[0] identifies whether the descriptor is valid, and is 1 for a valid descriptor. If a lookup returns an invalid descriptor, the associated input address is unmapped, and any attempt to access it generates a Translation fault.  
Descriptor bit[1] identifies the descriptor type, and is encoded as:

||||
| -- | -- | -- |
| **0** | **Block** | The descriptor gives the base address of a block of memory, and the attributes for that memory region. |
| **1** | **Table** | The descriptor gives the address of the next level of translation table, and for a stage 1 translation, some attributes for that translation. |

The other fields in the valid descriptors are:

**Block descriptor**  
Gives the base address and attributes of a block of memory, as follows:  

**4KB translation granule**  
* For a level 1 Block descriptor, bits[47:30] are bits[47:30] of the output address. This output address specifies a 1GB block of memory.
* For a level 2 descriptor, bits[47:21] are bits[47:21] of the output address.This output address specifies a 2MB block of memory.

**16KB translation granule**  
For a level 2 Block descriptor, bits[47:25] are bits[47:25] of the output address.This output address specifies a 32MB block of memory.  

**64KB translation granule**  
For a level 2 Block descriptor, bits[47:29] are bits[47:29] of the output address.This output address specifies a 512MB block of memory.
Bits[63:52, 11:2] provide attributes for the target memory block, see Memory attribute fields in the VMSAv8-64 translation table format descriptors on page D4-1699. The position and contents of these bits are identical in the level 2 block descriptor and in the level 3 page descriptor.


























