## D4.3.2 ARMv8 translation table level 3 descriptor formats

使用 4KB granule 时，level 3 table 中的每一个 entry 都映射了 4KB 的 input address range。  
使用 16KB granule 时，level 3 table 中的每一个 entry 都映射了 16KB 的 input address range。  
使用 64KB granule 时，level 3 table 中的每一个 entry 都映射了 64KB 的 input address range。
Figure D4-17 描述了 ARMv8 level 3 descriptor 的格式。

![](figure_d4_17.png)

Descriptor 的 bit[0] 指明该 descriptor 是否有效，该 bit 为 1 时，为有效的 descriptor。如果一次 lookup 操作返回一个无效的 descriptor，那么就意味着该 input address 没有进行映射，当访问该 input address 时，会产生 Translation fault。  

Descriptor 的 bit[1] 指明该 descriptor 的类型，如下所示：

||||
| -- | -- | -- |
| **0** | **Reserved, invalid** | 与 bit[0] 为 0 时的效果一样，指示 Descriptor 为无效的。 |
| **1** | **Page** | Descriptor 包含了 4KB、16KB 或者 64KB page 的地址和属性信息。|

在此 level 中，只存在 Page descriptor。Page descriptor 的其他位的含义如下：

**Page descriptor**  
该 descriptor 包含了一个 page 的地址，如下所示：
**4KB translation granule**  
Bits[47:12] 为 page 的地址的 bits[47:12]。  
**16KB translation granule**  
Bits[47:14] 为 page 的地址的 bits[47:14]。  
**64KB translation granule**  
Bits[47:16] 为 page 的地址的 bits[47:16]。   
Bits[63:52, 11:2] 包含 page 的属性信息，更多细节参考 [Memory attribute fields in the VMSAv8-64 translation table format descriptors](#) 章节.

> **NOTE**:  
> The position and contents of bits[63:52, 11:2] are identical to bits[63:52, 11:2] in the level 0, level 1, and level 2 block descriptors.

如果 translation table 属于 Non-secure EL1&0 stage 1 translation，那么 descriptor 中的 output address 为 page 的 IPA，如果不属于，那么 output address 为 page 的 PA。
