2026-04-04 22:38

Tags: [[ALU]] [[arithmetic operations]] [[binary]] [[floats]] 

### Arithmetic Operations

Binary addition i just the same idea as adding in decimal but it's base 2 instead because we're in binary. So the only available values are 0 and 1. You essentially just add bit by bit from right to left. Carry's work the same way. 

For subtraction it's a bit different since computers can't really subtract. For subtraction you have to convert the number to Two's Complement which is just inverting all the bits and adding 1. Then you can add the bits together like in addition.

Overflow can happen when the result of the bits doesn't fit in the fixed number of bits. 

The ALU (Arithmetic Logic Unit) performs these operations as well as logical operations like AND, OR, etc. 

Multiplication is a bit different, you have to shift the bits as you compute and then add them. Division also works pretty similar to the long division that we're used to. It also may have remainders.
