># D4.1 About the Virtual Memory System Architecture (VMSA)

> This chapter describes the Virtual Memory System Architecture (VMSA) 
 > that applies to a PE executing in AArch64 state.
> This is VMSAv8-64, as defined in [ARMv8 VMSA naming on page D4-1643](todo.md).


> A VMSA provides a Memory Management Unit (MMU), 
 > that controls address translation, access permissions,
 > and memory attribute determination and checking, 
 > for memory accesses made by the PE.


> The process of address translation maps the virtual addresses (VAs) used by 
 > PE onto the physical addresses (PAs) of the physical memory system. 
> These translations are defined independently for different Exception levels
 > and Security states, and Figure D4-1 shows.

