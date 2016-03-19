## D4.3.3 Memory attribute fields in the VMSAv8-64 translation table format descriptors

[Memory region attributes on page D4-1712](#) describes the region attribute fields. The following subsections summarize the descriptor attributes as follows:  
**Table descriptor**  
Table descriptors for stage 2 translations do not include any attribute field. For a summary of the attribute fields in a stage 1 table descriptor, that define the attributes for the next lookup level, see Next-level attributes in stage 1 VMSAv8-64 Table descriptors.  
**Block and page descriptors**  
These descriptors define memory attributes for the target block or page of memory. Stage 1 and stage 2 translations have some differences in these attributes, see:  
* Attribute fields in stage 1 VMSAv8-64 Block and Page descriptors on page D4-1700
* Attribute fields in stage 2 VMSAv8-64 Block and Page descriptors on page D4-1701.

### Next-level attributes in stage 1 VMSAv8-64 Table descriptors

In a Table descriptor for a stage 1 translation, bits[63:59] of the descriptor define the attributes for the next-level translation table access, and bits[58:52] are IGNORED:

![](figure_d4_18.png)

