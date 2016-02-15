# D4.2 The VMSAv8-64 address translation system

This section describes the VMSAv8-64 address translation system, that maps VAs to PAs. Related to this:  
 * [VMSAv8-64 translation table format descriptors on page D4-1695](#) describes the translation table entries.
 * [Access controls and memory region attributes on page D4-1704](#) describes the attributes that are held in the
   translation table entries, including how different attributes can interact.
 * Translation Lookaside Buffers (TLBs) on page D4-1729 describes the caching of translation table lookups in
   TLBs, and the architected instructions for maintaining TLBs.
 * AArch64 Address translation examples on page J7-5480 gives detailed descriptions of typical examples of
   translating a VA to a final PA, and obtaining the memory attributes of that PA.
