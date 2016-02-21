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

```
// AArch64.FirstStageTranslate()
// =============================
// Perform a stage 1 translation walk. The function used by Address Translation operations is 
// similar except it uses the translation regime specified for the instruction.

AddressDescriptor AArch64.FirstStageTranslate(bits(64) vaddress, AccType acctype, boolean iswrite,                                                boolean wasaligned, integer size)
    if HaveEL(EL2) && !IsSecure() && PSTATE.EL IN {EL0,EL1} then
        s1_enabled = HCR_EL2.TGE == '0' && HCR_EL2.DC == '0' && SCTLR_EL1.M == '1';
    else
        s1_enabled = SCTLR[].M == '1';
        
    ipaddress = bits(48) UNKNOWN; 
    secondstage = FALSE;
    s2fs1walk = FALSE;
    
    if s1_enabled then                         // First stage enabled
        S1 = AArch64.TranslationTableWalk(ipaddress, vaddress, acctype, iswrite, secondstage,
                                          s2fs1walk, size);
        permissioncheck = TRUE;
    else
        S1 = AArch64.TranslateAddressS1Off(vaddress, acctype, iswrite); 
        permissioncheck = FALSE;
    
    // Check for unaligned data accesses to Device memory
    if (!wasaligned && !IsFault(S1.addrdesc) && S1.addrdesc.memattrs.type == MemType_Device &&
        acctype != AccType_IFETCH) then
        S1.addrdesc.fault = AArch64.AlignmentFault(acctype, iswrite, secondstage);
        
    if !IsFault(S1.addrdesc) && permissioncheck then
        S1.addrdesc.fault = AArch64.CheckPermission(S1.perms, vaddress, S1.level,
                                                    S1.addrdesc.paddress.NS,
                                                    acctype, iswrite);
                                                    
    
    // Check for instruction fetches from Device memory not marked as execute-never. If there has
    // not been a Permission Fault then the memory is not marked execute-never.
    if (!IsFault(S1.addrdesc) && S1.addrdesc.memattrs.type == MemType_Device &&
        acctype == AccType_IFETCH) then
        S1.addrdesc = AArch64.InstructionDevice(S1.addrdesc, vaddress, ipaddress, S1.level,
                                                acctype, iswrite,
                                                secondstage, s2fs1walk);
    
    return S1.addrdesc;
```

When stage 1 translation is disabled, the function AArch64.TranslateAddressS1Off() sets the memory attributes.

```
// AArch64.TranslateAddressS1Off()
// ===============================
// Called for stage 1 translations when translation is disabled to supply a default translation.
// Note that there are additional constraints on instruction prefetching that are not described in
// this pseudocode.

TLBRecord AArch64.TranslateAddressS1Off(bits(64) vaddress, AccType acctype, boolean iswrite)
    assert !ELUsingAArch32(S1TranslationRegime());
    
    TLBRecord result;
    
    Top = AddrTop(vaddress);
    if !IsZero(vaddress<Top:PAMax()>) then
        level = 0;
        ipaddress = bits(48) UNKNOWN;
        secondstage = FALSE;
        s2fs1walk = FALSE;
        result.addrdesc.fault = AArch64.AddressSizeFault(ipaddress, level, acctype,
                                                         iswrite, secondstage, s2fs1walk);
        return result;
    
    default_cacheable = (HaveEL(EL2) && !IsSecure() && PSTATE.EL IN {EL0,EL1} && HCR_EL2.DC == '1');
    
    if default_cacheable then
        // Use default cacheable settings 
        result.addrdesc.memattrs.type = MemType_Normal; 
        result.addrdesc.memattrs.inner.attrs = MemAttr_WB;   // Write-back
        result.addrdesc.memattrs.inner.hints = MemHint_RWA; 
        result.addrdesc.memattrs.shareable = FALSE; 
        result.addrdesc.memattrs.outershareable = FALSE;
    elsif acctype != AccType_IFETCH then
        // Treat data as Device
        result.addrdesc.memattrs.type = MemType_Device; 
        result.addrdesc.memattrs.device = DeviceType_nGnRnE; 
        result.addrdesc.memattrs.inner = MemAttrHints UNKNOWN;
    else
        // Instruction cacheability controlled by SCTLR_ELx.I 
        cacheable = SCTLR[].I == '1'; 
        result.addrdesc.memattrs.type = MemType_Normal;
        if cacheable then
            result.addrdesc.memattrs.inner.attrs = MemAttr_WT;
            result.addrdesc.memattrs.inner.hints = MemHint_RA;
        else
            result.addrdesc.memattrs.inner.attrs = MemAttr_NC;
            result.addrdesc.memattrs.inner.hints = MemHint_No;
        result.addrdesc.memattrs.shareable = TRUE; 
        result.addrdesc.memattrs.outershareable = TRUE;
    
    result.addrdesc.memattrs.outer = result.addrdesc.memattrs.inner;
    
    result.addrdesc.memattrs = MemAttrDefaults(result.addrdesc.memattrs);
    
    result.perms.ap = bits(3) UNKNOWN; 
    result.perms.xn = '0'; 
    result.perms.pxn = '0';
    
    result.nG = bit UNKNOWN;
    result.contiguous = boolean UNKNOWN;
    result.domain = bits(4) UNKNOWN;
    result.level = integer UNKNOWN;
    result.blocksize = integer UNKNOWN; 
    result.addrdesc.paddress.physicaladdress = vaddress<47:0>; 
    result.addrdesc.paddress.NS = if IsSecure() then '0' else '1'; 
    result.addrdesc.fault = AArch64.NoFault();
    
    return result;
```

### Stage 2 translation

In the Non-secure EL1&0 translation regime, a descriptor address returned by stage 1 lookup is in the IPA address space, and must be mapped to a PA by a stage 2 translation. Function AArch64.SecondStageWalk() performs this translation, by calling the AArch64.SecondStageTranslate() function.When called from AArch64.SecondStageWalk(), the AArch64.SecondStageTranslate() function performs a second stage translation, from IPA to PA, of the supplied address, including checking that the access has read permission at the second stage. If the access does not have second stage read permission it generates a second stage Permission fault on the first stage translation table walk. The second stage translation might hit in a TLB, or might involve a translation table walk, which will use the algorithm described in this section.

```
// AArch64.SecondStageWalk()
// =========================
// Perform a stage 2 translation on a stage 1 translation page table walk access.

AddressDescriptor AArch64.SecondStageWalk(AddressDescriptor S1, bits(64) vaddress, AccType acctype,
                                          boolean iswrite, integer size)
    assert HaveEL(EL2) && !IsSecure() && PSTATE.EL IN {EL0,EL1};
    
    s2fs1walk = TRUE;
    wasaligned = TRUE;
    return AArch64.SecondStageTranslate(S1, vaddress, acctype, iswrite, wasaligned, s2fs1walk,
                                        size);
```

The AArch64.SecondStageTranslate() function performs the stage 2 address translation.

```
// AArch64.SecondStageTranslate()
// ==============================
// Perform a stage 2 translation walk. The function used by Address Translation operations is 
// similar except it uses the translation regime specified for the instruction.

AddressDescriptor AArch64.SecondStageTranslate(AddressDescriptor S1, bits(64) vaddress,
                                               AccType acctype, boolean iswrite, boolean wasaligned,
                                               boolean s2fs1walk, integer size)
    assert HaveEL(EL2) && !IsSecure() && PSTATE.EL IN {EL0,EL1};
    
    s2_enabled = HCR_EL2.VM == '1' || HCR_EL2.DC == '1'; 
    secondstage = TRUE;
    
    if s2_enabled then // Second stage enabled
        ipaddress = S1.paddress.physicaladdress<47:0>;
        S2 = AArch64.TranslationTableWalk(ipaddress, vaddress, acctype, iswrite, secondstage,
                                          s2fs1walk, size);

        // Check for unaligned data accesses to Device memory
        if (!wasaligned && !IsFault(S2.addrdesc) && S2.addrdesc.memattrs.type == MemType_Device &&
            acctype != AccType_IFETCH) then
            S2.addrdesc.fault = AArch64.AlignmentFault(acctype, iswrite, secondstage);
            
        if !IsFault(S2.addrdesc) then
            S2.addrdesc.fault = AArch64.CheckS2Permission(S2.perms, vaddress, ipaddress, S2.level,
                                                          acctype, iswrite, s2fs1walk);
    
        // Check for instruction fetches from Device memory not marked as execute-never. As there 
        // has not been a Permission Fault then the memory is not marked execute-never.
        if (!IsFault(S2.addrdesc) && S2.addrdesc.memattrs.type == MemType_Device &&
            acctype == AccType_IFETCH) then
            S2.addrdesc = AArch64.InstructionDevice(S2.addrdesc, vaddress, ipaddress, S2.level,
                                                    acctype, iswrite, 
                                                    secondstage, s2fs1walk);
    
        // Check for protected table walk
        if (s2fs1walk && !IsFault(S2.addrdesc) && HCR_EL2.PTW == '1' &&
            S2.addrdesc.memattrs.type == MemType_Device) then
            S2.addrdesc.fault = AArch64.PermissionFault(ipaddress, S2.level, acctype,
                                                        iswrite, secondstage, s2fs1walk);
        result = CombineS1S2Desc(S1, S2.addrdesc);

    else
        result = S1;
    return result;

```

### Translation table walk

The function AArch64.TranslationTableWalk() returns the result, in the form of a TLBRecord, of a translation table walk made for a memory access from an Exception level that is using AArch64.

```
// AArch64.TranslationTableWalk()
// ==============================
// Returns a result of a translation table walk 
//
// Implementations might cache information from memory in any number of non-coherent TLB
// caching structures, and so avoid memory accesses that have been expressed in this
// pseudocode. The use of such TLBs is not expressed in this pseudocode.

TLBRecord AArch64.TranslationTableWalk(bits(48) ipaddress, bits(64) vaddress,
                                       AccType acctype, boolean iswrite, boolean secondstage,
                                       boolean s2fs1walk, integer size)

    if !secondstage then
        assert !ELUsingAArch32(S1TranslationRegime());
    else
        assert HaveEL(EL2) && !IsSecure() && !ELUsingAArch32(EL2) && PSTATE.EL != EL2;

    TLBRecord result;
    AddressDescriptor descaddr;
    bits(64) baseregister;
    bits(64) inputaddr; // Input Address is 'vaddress' for stage 1, 'ipaddress' for stage 2
    
    descaddr.memattrs.type = MemType_Normal;

    // Derived parameters for the page table walk:
    // grainsize = Log2(Size of Table)         - Size of Table is 4KB, 16KB or 64KB in AArch64 
    // stride = Log2(Address per Level)        - Bits of address consumed at each level
    // firstblocklevel = First level where a block entry is allowed
    // ps = Physical Address size as encoded in TCR_EL1.IPS or TCR_ELx/VTCR_EL2.PS
    // inputsize = Log2(Size of Input Address) - Input Address size in bits
    // level = Level to start walk from
    // This means that the number of levels after start level = 3-level

```






