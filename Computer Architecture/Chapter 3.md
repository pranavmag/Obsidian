2026-04-04 22:38

Tags: [[ALU]] [[arithmetic operations]] [[binary]] [[floats]] 

### Arithmetic Operations

Binary addition i just the same idea as adding in decimal but it's base 2 instead because we're in binary. So the only available values are 0 and 1. You essentially just add bit by bit from right to left. Carry's work the same way. 

For subtraction it's a bit different since computers can't really subtract. For subtraction you have to convert the number to Two's Complement which is just inverting all the bits and adding 1. Then you can add the bits together like in addition.

Overflow can happen when the result of the bits doesn't fit in the fixed number of bits. 

The ALU (Arithmetic Logic Unit) performs these operations as well as logical operations like AND, OR, etc. 

Multiplication is a bit different, you have to shift the bits as you compute and then add them. Division also works pretty similar to the long division that we're used to. It also may have remainders.



### Floating-Point Representation

Floating-Point Representation has standards that were released by IEE, and the two most common are single precision (32 bit) and double precision (64 bit). 

In single precision 1 bit is given to the sign, 8 bits are given to the exponent, and 23 bits are given to the mantissa. In double precision 1 bit is given to the sign, 11 bits and given to the exponent, and 52 bits are given to the mantissa. The mantissa getting more bits allows us to have more precision in double precision and the exponent getting a couple more bits allows us to have a larger range of values.

Adding floating point values is not too complex. You have to match the significands decimal points to the one with the highest exponent. So for example, if you have 1.000 x 2^-1 + -1.110 x 2^-2 you have to shift the right significand and make it -0.111 x 2^-1 to match the other significand with the higher exponent. Then you add the significands together, and then normalize the result for over/underflow, and then round and renormalize if necessary. You have to do this if the amount of digits you have goes over the amount of digits you can have. 
