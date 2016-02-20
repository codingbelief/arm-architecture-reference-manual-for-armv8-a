## D4.2.10 Pseudocode description of VMSAv8-64 address translation

The following subsections gives a pseudocode description of the translation table walk:
* [Definitions required for address translation](#).
* [Performing the full address translation](#).
* [Stage 1 translation on page D4-1681](#).
* [Stage 2 translation on page D4-1683](#).
* [Translation table walk on page D4-1684](#).
* [Support functions on page D4-1689](#).

### Definitions required for address translation

In pseudocode, the result of a translation table lookup, in either Execution state, is returned in a TLBRecord structure.
```
type TLBRecord is (
    Permissions       perms,
    bit               nG,         // '0' = Global, '1' = not Global
    bits(4)           domain,     // AArch32 only
    boolean           contiguous, // Contiguous bit from page table
    integer           level,      // In AArch32 Short-descriptort format, indicates Section/Page
    integer           blocksize,  // Describes size of memory translated in KBytes
    AddressDescriptor addrdesc
)
```

[Memory data type definitions on page D3-1626](#) includes definitions of the Permissions and AddressDescriptor parameters.

### Performing the full address translation

The function AArch64.FullTranslate() performs a full translation table walk. For any translation regime it performs a stage 1 translation for the supplied virtual address, and for the Non-secure EL1&0 translation regime it then performs a stage 2 translation of the returned address.

