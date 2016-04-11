## D4.4.1 Memory access control

The access control fields in the translation table descriptors determine whether the PE, in its current state, is permitted to perform the required access to the output address given in the translation table descriptor. If a translation stage does not permit the access then an MMU fault is generated for that translation stage, and no memory access is performed.  
The following sections describe the memory access controls:
* [About the access permissions](#).
* [The data access permission controls on page D4-1705](#).
* [Access permissions for instruction execution on page D4-1707](#).
* The Access flag on page D4-1711.

### About the access permissions

