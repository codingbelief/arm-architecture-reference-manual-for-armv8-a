# D4 The AArch64 Virtual Memory System Architecture

>This chapter provides a system level view of the AArch64 Virtual Memory System Architecture (VMSA),
>the memory system architecture of an ARMv8 implementation that is executing in AArch64 state. 
>It contains the following sections:
>   * [D4.1 About the Virtual Memory System Architecture (VMSA)](d41_about_the_virtual_memory_system_architecture_v_.md)
>   * [D4.2 The VMSAv8-64 address translation system](d42_the_vmsav8-64_address_translation_system.md)
>   * [D4.3 VMSAv8-64 translation table format descriptors](d43_vmsav8-64_translation_table_format_descriptors.md)
>   * [D4.4 Access controls and memory region attributes](d44_access_controls_and_memory_region_attributes.md)
>   * [D4.5 MMU faults](d45_mmu_faults.md)
>   * [D4.6 Translation Lookaside Buffers (TLBs)](d46_translation_lookaside_buffers_tlbs.md)
>   * [D4.7 TLB maintenance requirements and the TLB maintenance instructions](d47_tlb_maintenance_requirements_and_the_tlb_maint.md)
>   * [D4.8 Caches in a VMSA implementation](d48_caches_in_a_vmsa_implementation.md)

此章节在系统层面上介绍了 ARMv8 在 AArch64 运行态下的内存系统架构的实现，
即 VMSA ( AArch64 Virtual Memory System Architecture )。
主要包括以下小节：

   * [D4.1 VMSA 简介](d41_about_the_virtual_memory_system_architecture_v_.md)
   * [D4.2 VMSAv8-64 地址转换系统](d42_the_vmsav8-64_address_translation_system.md)
   * [D4.3 VMSAv8-64 地址转换表](d43_vmsav8-64_translation_table_format_descriptors.md)
   * [D4.4 访问控制和内存区域属性](d44_access_controls_and_memory_region_attributes.md)
   * [D4.5 MMU 异常](d45_mmu_faults.md)
   * [D4.6 Translation Lookaside Buffers (TLBs)](d46_translation_lookaside_buffers_tlbs.md)
   * [D4.7 TLB 操作需求和操作指令](d47_tlb_maintenance_requirements_and_the_tlb_maint.md)
   * [D4.8 VMSA 中的 Caches 实现](d48_caches_in_a_vmsa_implementation.md)