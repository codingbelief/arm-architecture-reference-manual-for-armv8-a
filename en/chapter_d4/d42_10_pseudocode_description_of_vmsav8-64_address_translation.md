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
    Permissions perms,
    bit nG,
    bits(4) domain, 
    boolean contiguous, 
    integer level, 
    integer blocksize, 
    AddressDescriptor addrdesc
// '0' = Global, '1' = not Global
// AArch32 only
// Contiguous bit from page table
// In AArch32 Short-descriptort format, indicates Section/Page // Describes size of memory translated in KBytes
)
```