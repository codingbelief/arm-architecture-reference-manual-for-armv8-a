## D4.2.11 Address translation instructions

Each of the ARMv8 instruction sets provides instructions that return the result of translating an input address, supplied as an argument to the instruction, using a specified translation stage or regime.

The available instructions only perform translations that are accessible from the Security state and Exception level at which the instruction is executed. That is:
* No instruction executed in Non-secure state can return the result of a Secure address translation stage.
* No instruction can return the result of an address translation stage that is controlled by an Exception level that is higher than the Exception level at which the instruction is executed.

[Address translation instructions, AT* on page D4-1692](#) summarizes the A64 address translation instructions. 

See also [A64 system instructions for address translation on page C5-334](#).

### Address translation instructions, AT*


