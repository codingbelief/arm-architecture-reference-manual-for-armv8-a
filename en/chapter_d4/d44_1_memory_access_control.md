## D4.4.1 Memory access control

The access control fields in the translation table descriptors determine whether the PE, in its current state, is permitted to perform the required access to the output address given in the translation table descriptor. If a translation stage does not permit the access then an MMU fault is generated for that translation stage, and no memory access is performed.  
The following sections describe the memory access controls:
* [About the access permissions](#).
* [The data access permission controls on page D4-1705](#).
* [Access permissions for instruction execution on page D4-1707](#).
* [The Access flag on page D4-1711](#).

### About the access permissions

> **NOTE:**  
> This section gives a general description of memory access permissions. In an implementation that includes EL2, software executing at EL1 in Non-secure state can see only the access permissions defined by the Non-secure EL1&0 stage 1 translations. However, software executing at EL2 can modify these permissions. This modification is invisible to the Non-secure software executing at EL1 or EL0.

The access permission bits control access to the corresponding memory region. The VMSAv8-64 translation table format:
* In stage 1 translations, uses AP[2:1] to define the data access permissions, see The AP[2:1] data access permissions, for stage 1 translations on page D4-1705.
 > **NOTE:**  
 The description of the access permission field as AP[2:1] is for consistency with the VMSAv8-32 Short-descriptor translation table format, see The VMSAv8-32 Short-descriptor translation table format on page G4-4015. The VMSAv8-64 translation table format does not define an AP[0] bit.