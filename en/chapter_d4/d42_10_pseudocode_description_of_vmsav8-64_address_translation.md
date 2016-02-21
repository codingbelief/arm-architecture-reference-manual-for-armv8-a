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

    if !secondstage then
        // First stage translation 
        inputaddr = ZeroExtend(vaddress); 
        top = AddrTop(inputaddr);
        
        if PSTATE.EL == EL3 then
            inputsize = 64 - UInt(TCR_EL3.T0SZ); 
            if inputsize > 48 then
                c = ConstrainUnpredictable();
                assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                if c == Constraint_FORCE then inputsize = 48;
            if inputsize < 25 then
                c = ConstrainUnpredictable();
                assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                if c == Constraint_FORCE then inputsize = 25;
            largegrain = TCR_EL3.TG0 == '01';
            midgrain = TCR_EL3.TG0 == '10';
            ps = TCR_EL3.PS;
            basefound = inputsize >= 25 && inputsize <= 48 && IsZero(inputaddr<top:inputsize>); disabled = FALSE;
            baseregister = TTBR0_EL3;
            descaddr.memattrs = WalkAttrDecode(TCR_EL3.SH0, TCR_EL3.ORGN0, TCR_EL3.IRGN0); reversedescriptors = SCTLR_EL3.EE == '1';
            lookupsecure = TRUE;
            singlepriv = TRUE;

        elsif PSTATE.EL == EL2 then
            inputsize = 64 - UInt(TCR_EL2.T0SZ); 
            if inputsize > 48 then
                c = ConstrainUnpredictable();
                assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                if c == Constraint_FORCE then inputsize = 48;
            if inputsize < 25 then
                c = ConstrainUnpredictable();
                assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                if c == Constraint_FORCE then inputsize = 25;
            largegrain = TCR_EL2.TG0 == '01';
            midgrain = TCR_EL2.TG0 == '10';
            ps = TCR_EL2.PS;
            basefound = inputsize >= 25 && inputsize <= 48 && IsZero(inputaddr<top:inputsize>); disabled = FALSE;
            baseregister = TTBR0_EL2;
            descaddr.memattrs = WalkAttrDecode(TCR_EL2.SH0, TCR_EL2.ORGN0, TCR_EL2.IRGN0); reversedescriptors = SCTLR_EL2.EE == '1';
            lookupsecure = FALSE;
            singlepriv = TRUE;
            
        else
            if inputaddr<top> == '0' then
                inputsize = 64 - UInt(TCR_EL1.T0SZ); 
                if inputsize > 48 then
                    c = ConstrainUnpredictable();
                    assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                    if c == Constraint_FORCE then inputsize = 48;
                if inputsize < 25 then
                    c = ConstrainUnpredictable();
                    assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                    if c == Constraint_FORCE then inputsize = 25;
                largegrain = TCR_EL1.TG0 == '01';
                midgrain = TCR_EL1.TG0 == '10';
                basefound = inputsize >= 25 && inputsize <= 48 && IsZero(inputaddr<top:inputsize>); disabled = TCR_EL1.EPD0 == '1';
                baseregister = TTBR0_EL1;
                descaddr.memattrs = WalkAttrDecode(TCR_EL1.SH0, TCR_EL1.ORGN0, TCR_EL1.IRGN0);
            else
                inputsize = 64 - UInt(TCR_EL1.T1SZ); 
                if inputsize > 48 then
                    c = ConstrainUnpredictable();
                    assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                    if c == Constraint_FORCE then inputsize = 48;
                if inputsize < 25 then
                    c = ConstrainUnpredictable();
                    assert c IN {Constraint_FORCE, Constraint_FAULT}; 
                    if c == Constraint_FORCE then inputsize = 25;
                largegrain = TCR_EL1.TG1 == '11'; // TG1 and TG0 encodings differ
                midgrain = TCR_EL1.TG1 == '01';
                basefound = inputsize >= 25 && inputsize <= 48 && IsOnes(inputaddr<top:inputsize>); disabled = TCR_EL1.EPD1 == '1';
                baseregister = TTBR1_EL1;
                descaddr.memattrs = WalkAttrDecode(TCR_EL1.SH1, TCR_EL1.ORGN1, TCR_EL1.IRGN1);
            ps = TCR_EL1.IPS;
            reversedescriptors = SCTLR_EL1.EE == '1'; 
            lookupsecure = IsSecure();
            singlepriv = FALSE;

        if largegrain then 
            grainsize = 16;       // Log2(64KB page size)
            firstblocklevel =2;   // Largest block is 512MB (2^29 bytes)
        elsif midgrain then
            grainsize = 14;       // Log2(16KB page size)
            firstblocklevel =2;   // Largest block is 32MB (2^25 bytes)
        else // Small grain
            grainsize = 12;       // Log2(4KB page size)
            firstblocklevel = 1;  // Largest block is 1GB (2^30 bytes) 
        stride = grainsize - 3;   // Log2(page size / 8 bytes)
        // The starting level is the number of strides needed to consume the input address 
        level = 4 - RoundUp(Real(inputsize - grainsize) / Real(stride));
        
    else
        // Second stage translation
        inputaddr = ZeroExtend(ipaddress); 
        inputsize = 64 - UInt(VTCR_EL2.T0SZ); 
        if inputsize > 48 then
            c = ConstrainUnpredictable();
            assert c IN {Constraint_FORCE, Constraint_FAULT}; 
            if c == Constraint_FORCE then inputsize = 48;
        if inputsize < 25 then
            c = ConstrainUnpredictable();
            assert c IN {Constraint_FORCE, Constraint_FAULT}; 
            if c == Constraint_FORCE then inputsize = 25;
        largegrain = VTCR_EL2.TG0 == '01';
        midgrain = VTCR_EL2.TG0 == '10';
        ps = VTCR_EL2.PS;
        basefound = inputsize >= 25 && inputsize <= 48 && IsZero(inputaddr<63:inputsize>); 
        disabled = FALSE;
        baseregister = VTTBR_EL2;
        descaddr.memattrs = WalkAttrDecode(VTCR_EL2.IRGN0, VTCR_EL2.ORGN0, VTCR_EL2.SH0); reversedescriptors = SCTLR_EL2.EE == '1';
        lookupsecure = FALSE;
        singlepriv = TRUE;

        startlevel = UInt(VTCR_EL2.SL0); 
        if largegrain then
            grainsize = 16;                 // Log2(64KB page size)
            level = 3 - startlevel; 
            firstblocklevel = 2;            // Largest block is 512MB (2^29 bytes)
        elsif midgrain then 
            grainsize = 14;                 // Log2(16KB page size)
            level = 3 - startlevel;
            firstblocklevel = 2;            // Largest block is 32MB (2^25 bytes)
        else // Small grain
            grainsize = 12;                 // Log2(4KB page size)
            level = 2 - startlevel; 
            firstblocklevel = 1;            // Largest block is 1GB (2^30 bytes)
        stride = grainsize - 3;             // Log2(page size / 8 bytes)

        // Limits on IPA controls based on implemented PA size. Level 0 is only 
        // supported by small grain translations
        if largegrain then                      // 64KB pages
            // Level 1 only supported if implemented PA size is greater than 2^42 bytes
            if level == 0 || (level == 1 && PAMax() <= 42) then basefound = FALSE; 
        elsif midgrain then                     // 16KB pages
            // Level 1 only supported if implemented PA size is greater than 2^40 bytes
            if level == 0 || (level == 1 && PAMax() <= 40) then basefound = FALSE; 
        else                                    // Small grain, 4KB pages
            // Level 0 only supported if implemented PA size is greater than 2^42 bytes 
            if level < 0 || (level == 0 && PAMax() <= 42) then basefound = FALSE;

        // If the inputsize exceeds the PAMax value, the behavior is CONSTRAINED UNPREDICTABLE
        inputsizecheck = inputsize;
        if inputsize > PAMax() && (!ELUsingAArch32(EL1) || inputsize > 40) then
            case ConstrainUnpredictable() of 
                when Constraint_FORCE
                    // Restrict the inputsize to the PAMax value 
                    inputsize = PAMax();
                    inputsizecheck = PAMax();
                when Constraint_FORCENOSLCHECK
                    // As FORCE, except use the configured inputsize in the size checks below
                    inputsize = PAMax();
                when Constraint_FAULT
                    // Generate a translation fault
                    basefound = FALSE; 
                otherwise
                    Unreachable();
        
        // Number of entries in the starting level table =
        // (Size of Input Address)/((Address per level)^(Num levels remaining)*(Size of Table)) startsizecheck = inputsizecheck - ((3 - level)*stride + grainsize); // Log2(Num of entries)

        // Check for starting level table with fewer than 2 entries or longer than 16 pages. 
        // Lower bound check is: startsizecheck < Log2(2 entries)
        // Upper bound check is: startsizecheck > Log2(pagesize/8*16)
        if startsizecheck < 1 || startsizecheck > stride + 4 then basefound = FALSE;

    if !basefound || disabled then
        level = 0; // AArch32 reports this as a level 1 fault 
        result.addrdesc.fault = AArch64.TranslationFault(ipaddress, level, acctype, iswrite,
                                                         secondstage, s2fs1walk);
        return result;

    case ps of 
        when '000'  outputsize = 32;
        when '001'  outputsize = 36;
        when '010'  outputsize = 40;
        when '011'  outputsize = 42;
        when '100'  outputsize = 44;
        when '101'  outputsize = 48;
        otherwise   outputsize = 48;
        
    if outputsize > PAMax() then outputsize = PAMax();
    
    if outputsize != 48 && !IsZero(baseregister<47:outputsize>) then
        level = 0;
        result.addrdesc.fault = AArch64.AddressSizeFault(ipaddress, level, acctype, iswrite,
                                                         secondstage, s2fs1walk);
        return result;

    // Bottom bound of the Base address is:
    //     Log2(8 bytes per entry)+Log2(Number of entries in starting level table)
    // Number of entries in starting level table =
    //     (Size of Input Address)/(Address pre level)^(Num level remaining)*(Size of Table))
    baselowerbound = 3 + inputsize - ((3-level)*stride + grainsize); // Log2(Num of entries*8)
    baseaddress = baseregister<47:baselowerbound>:Zeros(baselowerbound);

    ns_table = if lookupsecure then '0' else '1'; 
    ap_table = '00';
    xn_table = '0';
    pxn_table = '0';

    addrselecttop = inputsize - 1;

    repeat
        addrselectbottom = (3-level)*stride + grainsize;
        
        bits(48) index = ZeroExtend(inputaddr<addrselecttop:addrselectbottom>:'000'); descaddr.paddress.physicaladdress = baseaddress OR index; 
        descaddr.paddress.NS = ns_table;
        
        // If there are two stages of translation, then the first stage table walk addresses 
        // are themselves subject to translation
        if !HaveEL(EL2) || secondstage || IsSecure() || PSTATE.EL == EL2 then
            descaddr2 = descaddr;
        else
            descaddr2 = AArch64.SecondStageWalk(descaddr, vaddress, acctype, iswrite, 8); 
            // Check for a fault on the stage 2 walk
            if IsFault(descaddr2) then
                result.addrdesc.fault = descaddr2.fault; 
                return result;

        // Update virtual address for abort functions 
        descaddr2.vaddress = ZeroExtend(vaddress);

        desc = _Mem[descaddr2, 8, AccType_PTW];
        if reversedescriptors then desc = BigEndianReverse(desc);
        
        if desc<0> == '0' || (desc<1:0> == '01' && level == 3) then 
            // Fault (00), Reserved (10), or Block (01) at level 3
            result.addrdesc.fault = AArch64.TranslationFault(ipaddress, level, acctype, 
                                                             iswrite, secondstage, s2fs1walk);
            return result;
            
        // Valid Block, Page, or Table entry
        if desc<1:0> == '01' || level == 3 then                 // Block (01) or Page (11)
            blocktranslate = TRUE;                         
        else                                                    // Table (11)
            if outputsize != 48 && !IsZero(desc<47:outputsize>) then
                result.addrdesc.fault = AArch64.AddressSizeFault(ipaddress, level, acctype, 
                                                                 iswrite, secondstage, s2fs1walk);
                return result;
                
            baseaddress = desc<47:grainsize>:Zeros(grainsize);

            if !secondstage then
                // Unpack the upper and lower table attributes
                ns_table = ns_table OR desc<63>;
                ap_table<1> = ap_table<1> OR desc<62>;      // read-only
                xn_table = xn_table OR desc<60>;
                // pxn_table and ap_table[0] apply only in EL1&0 translation regimes 
                if !singlepriv then
                    ap_table<0> = ap_table<0> OR desc<61>;  // privileged
                    pxn_table = pxn_table OR desc<59>;

            level = level + 1;
            addrselecttop = addrselectbottom - 1; blocktranslate = FALSE;
    until blocktranslate;

    // Check block size is supported at this level 
    if level < firstblocklevel then
        result.addrdesc.fault = AArch64.TranslationFault(ipaddress, level, acctype, 
                                                         iswrite, secondstage, s2fs1walk);
        return result;

    // Check for misprogramming of the contiguous bit 
    if largegrain then
        contiguousbitcheck = level == 2 && inputsize < 34; 
    elsif midgrain then
        contiguousbitcheck = level == 2 && inputsize < 30; 
    else
        contiguousbitcheck = level == 1 && inputsize < 34;

    if contiguousbitcheck && desc<52> == '1' then
        if boolean IMPLEMENTATION_DEFINED "Translation fault on misprogrammed contiguous bit" then
            result.addrdesc.fault = AArch64.TranslationFault(ipaddress, level, acctype, 
                                                             iswrite, secondstage, s2fs1walk);
            return result;

    // Check the output address is inside the supported range 
    if outputsize != 48 && !IsZero(desc<47:outputsize>) then
        result.addrdesc.fault = AArch64.AddressSizeFault(ipaddress, level, acctype, 
                                                         iswrite, secondstage, s2fs1walk);
        return result;

    // Check the access flag 
    if desc<10> == '0' then
        result.addrdesc.fault = AArch64.AccessFlagFault(ipaddress, level, acctype, 
                                                        iswrite, secondstage, s2fs1walk);
        return result;

    // Unpack the descriptor into address and upper and lower block attributes 
    outputaddress = desc<47:addrselectbottom>:inputaddr<addrselectbottom-1:0>; 
    xn = desc<54>;
    pxn = desc<53>;
    contiguousbit = desc<52>; 
    nG = desc<11>;
    sh = desc<9:8>;
    ap = desc<7:6>:'1'; 
    memattr = desc<5:2>;                // AttrIndx and NS bit in stage 1

    result.domain = bits(4) UNKNOWN;    // Domains not used
    result.level = level;
    result.blocksize = 2^((3-level)*stride + grainsize);

    // Stage 1 translation regimes also inherit attributes from the tables 
    if !secondstage then
        result.perms.xn = xn OR xn_table;
        result.perms.ap<2> = ap<2> OR ap_table<1>; // Force read-only
        
        // PXN, nG and AP[1] apply only in EL1&0 stage 1 translation regimes 
        if !singlepriv then
            result.perms.ap<1> = ap<1> AND NOT(ap_table<0>); // Force privileged only
            result.perms.pxn = pxn OR pxn_table;
            // Pages from Non-secure tables are marked non-global in Secure EL1&0
            if IsSecure() then
                result.nG = nG OR ns_table; 
            else
                result.nG = nG; 
        else
            result.perms.ap<1> = '1';
            result.perms.pxn   = '0';
            result.nG          = '0';
        result.perms.ap<0> = '1';
        result.addrdesc.memattrs = AArch64.S1AttrDecode(sh, memattr<2:0>, acctype); result.addrdesc.paddress.NS = memattr<3> OR ns_table;
    else
        result.perms.ap<2:1> = ap<2:1>;
        result.perms.ap<0>   = '1';
        result.perms.xn      = xn;
        result.perms.pxn     = '0';
        result.nG            = '0';
        result.addrdesc.memattrs = S2AttrDecode(sh, memattr, acctype); 
        result.addrdesc.paddress.NS = '1';
        
    result.addrdesc.paddress.physicaladdress = outputaddress; 
    result.addrdesc.fault = AArch64.NoFault(); 
    result.contiguous = contiguousbit == '1';
    
    return result;

```

### Support functions

In the translation table walk functions, the WalkAttrDecode() function determines the attributes for a translation table lookup.

```
// WalkAttrDecode() 
// ================

MemoryAttributes WalkAttrDecode(bits(2) SH, bits(2) ORGN, bits(2) IRGN) 

    MemoryAttributes memattrs;
    
    AccType acctype = AccType_NORMAL;
    
    memattrs.type = MemType_Normal;
    memattrs.inner = ShortConvertAttrsHints(IRGN, acctype); 
    memattrs.outer = ShortConvertAttrsHints(ORGN, acctype); 
    memattrs.shareable = SH<1> == '1'; 
    memattrs.outershareable = SH == '10';

    return MemAttrDefaults(memattrs);
```

The function AArch64.S1AttrDecode() decodes the attributes from a stage 1 translation table lookup.

```
// AArch64.S1AttrDecode()
// ======================
// Converts the Stage 1 attribute fields, using the MAIR, to orthogonal 
// attributes and hints.

MemoryAttributes AArch64.S1AttrDecode(bits(2) SH, bits(3) attr, AccType acctype)

    MemoryAttributes memattrs;
    
    mair = MAIR[];
    index = 8 * UInt(attr); 
    attrfield = mair<index+7:index>;
    
    if ((attrfield<7:4> != '0000' && attrfield<3:0> == '0000') || 
        (attrfield<7:4> == '0000' && attrfield<3:0> != 'xx00')) then 
        // Reserved, maps to an allocated value
        (-, attrfield) = ConstrainUnpredictableBits();
    
    if attrfield<7:4> == '0000' then          // Device
        memattrs.type = MemType_Device; 
        case attrfield<3:0> of
            when '0000' memattrs.device = DeviceType_nGnRnE; 
            when '0100' memattrs.device = DeviceType_nGnRE; 
            when '1000' memattrs.device = DeviceType_nGRE;
            when '1100' memattrs.device = DeviceType_GRE;
            otherwise   Unreachable();         // Reserved, handled above

    elsif attrfield<3:0> != '0000' then        // Normal
        memattrs.type = MemType_Normal;
        memattrs.outer = LongConvertAttrsHints(attrfield<7:4>, acctype); 
        memattrs.inner = LongConvertAttrsHints(attrfield<3:0>, acctype); 
        memattrs.shareable = SH<1> == '1';
        memattrs.outershareable = SH == '10';

    else
        Unreachable();                         // Reserved, handled above
    
    return MemAttrDefaults(memattrs);

```

The function AArch64.CheckPermission() checks the access permissions returned by a stage 1 translation table lookup, see Access permission checking on page D3-1631.

The function AArch64.CheckS2Permission() checks the access permissions returned by a stage 2 translation table lookup.

```
// AArch64.CheckS2Permission()
// ===========================
// Function used for permission checking from AArch64 stage 2 translations

FaultRecord AArch64.CheckS2Permission(Permissions perms, bits(64) vaddress, bits(48) ipaddress,
                                      integer level, AccType acctype, boolean iswrite,
                                      boolean s2fs1walk)
    assert HaveEL(EL2) && !IsSecure() && !ELUsingAArch32(EL2) && PSTATE.EL != EL2;

    r = perms.ap<1> == '1'; 
    w = perms.ap<2> == '1'; 
    xn = perms.xn == '1';

    // Stage 1 walk is checked as a read, regardless of the original type
    if acctype == AccType_IFETCH && !s2fs1walk then 
        fail = xn;
    elsif iswrite && !s2fs1walk then 
        fail = !w;
        failedread = FALSE; 
    else
        fail = !r; 
        failedread = TRUE;
```




