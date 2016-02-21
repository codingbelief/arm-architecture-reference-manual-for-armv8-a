## D4.2.11 Address translation instructions

Each of the ARMv8 instruction sets provides instructions that return the result of translating an input address, supplied as an argument to the instruction, using a specified translation stage or regime.

The available instructions only perform translations that are accessible from the Security state and Exception level at which the instruction is executed. That is:
* No instruction executed in Non-secure state can return the result of a Secure address translation stage.
* No instruction can return the result of an address translation stage that is controlled by an Exception level that is higher than the Exception level at which the instruction is executed.

[Address translation instructions, AT* on page D4-1692](#) summarizes the A64 address translation instructions. 

See also [A64 system instructions for address translation on page C5-334](#).


### Address translation instructions, AT*

The A64 assembly language syntax for address translation instructions is:

```
    AT <operation>, <Xt>
```

Where:

```
<operation> Is one of S1E1R, S1E1W, S1E0R, S1E0W, S12E1R, S12E1W, S12E0R, S12E0W, S1E2R, S1E2W, S1E3R, or S1E3W.
            <operation> has a structure of <stages><level><read|write>, where: 
            <stages> Is one of:
                     S1 Stage 1 translation.
                     S12 Stage 1 translation followed by stage 2 translation.
            <level> Describes the Exception Level that the translation applies to. Is one of:
E0 EL0.
E1 EL1.
E2 EL2.
E3 EL3.
If <level> is higher than the current Exception Level the instruction is UNDEFINED.
<read|write>
Is one of:
R Read. W Write.
<Xt> The address to be translated. No alignment restrictions apply for the address.
```