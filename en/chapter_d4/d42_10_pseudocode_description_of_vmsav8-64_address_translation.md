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

```
// AArch64.FullTranslate()
// =======================
// Perform both stage 1 and stage 2 translation walks for the current translation regime. The 
// function used by Address Translation operations is similar except it uses the translation 
// regime specified for the instruction.

AddressDescriptor AArch64.FullTranslate(bits(64) vaddress, AccType acctype, boolean iswrite,

// First Stage Translation
S1 = AArch64.FirstStageTranslate(vaddress, acctype, iswrite, wasaligned, size);

if !IsFault(S1) && HaveEL(EL2) && !IsSecure() && PSTATE.EL IN {EL0,EL1} then
    s2fs1walk = FALSE;
    result = AArch64.SecondStageTranslate(S1, vaddress, acctype, iswrite, wasaligned, s2fs1walk,
                                          size);
else
    result = S1;

return result;
```

### Stage 1 translation

The function AArch64.FirstStageTranslate() performs a stage 1 translation, calling the function AArch64.TranslationTableWalk(), described in [Translation table walk on page D4-1684](#), to perform the required translation table walk. However, if stage 1 translation is disabled, it calls the function AArch64.TranslateAddressS1Off(), described in this section, to set the memory attributes.












