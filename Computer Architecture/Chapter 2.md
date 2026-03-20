2026-03-06 00:11

Tags: [[register operands]]

### Register Operands

A CPU performs operations using registers and can only do one operation at a time so we have register operands. For example,
add a, b, c takes b + c and sets that to a.
Assembly: add a, b, c 
C++: a = b + c;

A real example we have is f = (g + h) - (i + j).
add t0, g, h
add t1, i, j
sub f, t0, t1

t0 and t1 are temporary registers meanwhile f, g, h, i, and j are stored in saved registers.


You can also take the base address of arrays and access their individual elements by using an offset. 

A[12] = h + A[8]
h in x21, base address of A in x22

To access the 8th element you need an offset of 64 because we are using a doubleword which is 8 bytes so 8 x 8 = 64.

ld x9, 64(x22)   so here we are loading the value from memory address x22 with an offset of 64
add x9, x21, x9  and setting the new value of x9 as x21 + x9
sd x9, 96(x22)    and saving that value from x9 to memory address 96(x22) 

Registers are fast, memory is slow → load from memory, compute in registers, store back