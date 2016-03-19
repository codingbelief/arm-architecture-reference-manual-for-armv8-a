## D4.3.2 ARMv8 translation table level 3 descriptor formats

For the 4KB granule size, each entry in a level 3 table describes the mapping of the associated 4KB input address range.  
For the 16KB granule size, each entry in a level 3 table describes the mapping of the associated 16KB input address range.  
For the 64KB granule size, each entry in a level 3 table describes the mapping of the associated 64KB input address range.  
Figure D4-17 shows the ARMv8 level 3 descriptor formats.

![](figure_d4_17.png)
Descriptor bit[0] identifies whether the descriptor is valid, and is 1 for a valid descriptor. If a lookup returns an invalid descriptor, the associated input address is unmapped, and any attempt to access it generates a Translation fault.  
Descriptor bit[1] identifies the descriptor type, and is encoded as:  

**0, Reserved, invalid**  
Behaves identically to encodings with bit[0] set to 0.  
This encoding must not be used in level 3 translation tables.

**1, Page**  
