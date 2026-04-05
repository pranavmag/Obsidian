2026-03-06 00:11

Tags: [[register operands]] [[signed integers]] [[unsigned integers]] [[instructions]]
[[logical operations]] [[procedures]] [[functions]] [[if statements]] [[stack]] [[synchronization]]
[[threads]] [[processors]] [[atomic]]
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



### Signed and Unsigned Integers

Unsigned and signed integers both represent a range of values but the way they are represented is different. For example on a 32-bit system a 32-bit unsigned integer represents 2^32 values which is about 0 to 4 billion. Unsigned integers hold know signed bits so the range starts from 0 and so on. Signed integers however do have the leftmost bit represented a negative bit. For every positive bit there is a negative bit that complements it. So there are also about 4 billion values for a signed 32-bit integer but the range shifts downwards and represents those values from about -2 billion to 2 billion.



### Instructions

For RISC-V there are different types of instructions, essentially how these bits are split up and interpreted by the CPU depending on the type of instructions.

R-type instructions are used for instructions like adding and subtracting. The layout is something like

add x5, x6, x7

| funct7 | rs2 | rs1 | funct3 | rd | opcode |
| 7 bits  | 5    | 5    | 3         | 5   | 7           |

rs1 - source register 1 (first input x6)
rs2 - source register 2 (second input x7)
rd - destination register (stored result x5)
opcode - instruction category
funct7, funct3 - further specifies the operation
register -> register

I-type instructions are used for loading instructions like addi (Add Immediate) and ld (load doubleword). The layout is something like

addi x5, x6, 10

| immediate | rs1 | funct3 | rd | opcode |
| 12 bits       | 5    | 3         | 5   | 7           |

rs1 - source register 1 (first input x6)
immediate - value 10 (constant OR offset used with a base register)
rd - destination register (stored result x5)
opcode - instruction category
funct3 - further specifies the operation
The exact operation = opcode + funct3 + funct7
register -> immediate


S-type instructions are used for saving instructions like sd (store doubleword). It is similar to I-type instructions but the immediate is split to two locations because we need rs2 (the value to store) and rs1 (the base address). The layout is something like

sd x10, 80(x2)

imm[11:5] | rs2 | rs1 | funct3 | imm[4:0] | opcode |
| 7 bits      | 5    | 5    | 3         | 5             | 7           |

rs1 - source register 1 (base address x2)
rs2 - source register 2 (value being stored x10)
imm = 80 (offset of address x2)
opcode - instruction category
funct3 - further specifies the operation 

So in summary:
R-type → rd = rs1 op rs2

I-type → rd = rs1 op imm
        rd = Memory[rs1 + imm]

S-type → Memory[rs1 + imm] = rs2



### Logical Operations

if statements and loops in code under the hood can be used in assembly. There are statements like bne (branch if not equal), beq (branch if equal) and Loop: that allow us to use these statements.

Assembly code executes line by line so you have to have branches that you can automatically go to. For example:

bne x22, x23, Else
add x19, x20, x21
beq x0, x0, Exit
Else: sub x19, x20, x21
Exit: ...

In this example, the first statement executes if x22 != x23, which jumps over to the Else branch skipping over the following add and beq statements and automatically executing sub x19, x20, x21. But, if the first statement was not true meaning that x22 == x23, the code doesn't jump to the Else branch and continues on to the add statement. But you don't want it to eventually hit the Else branch so before the Else branch on the third statement we have a conditional that is always true, meaning if we make it there we don't hit the Else and skip over to the Exit.



### Procedures

A procedure is a jump to a different part of the program, where it does some work there, and then jumps back to where it came from. Every procedure call passes arguments, jumps to the function, allocates space on the stack, does work there, returns the result, and then jumps back to the caller.

For example, jal x1, function - jumps to the function and saves the return address in x1. The return will be using jalr x0, 0(x1) which jumps back using x1.

The stack is where we store things temporarily. It grows downward from high addresses to low addresses. You push on the stack to save new things and pop when you want to restore them to use.

The caller is the one who calls the function and they must save temporaries if needed later and argument registers. The callee is the function that's being called and must save the return address and saved registers (x8-x9, x18-x27).



### Synchronization

Multiple threads/processors share memory so the problem is that the timing is off when one thread writes and another thread reads. You'll get data races if two threads access the same memory.

This is why we use atomic operations which is an operation that cannot be interrupted halfway.

Thread A: reads 
Thread B: writes
Thread A: writes
Thread B: writes

This is dangerous because both threads think they acquired the lock. 

One solution is if memory = 0 a thread can get the lock but if memory = 1 then another thread already has it.

In RISC-V there are two instructions: lr.w (Load Reserved) that reads the value and watches that memory location and sc.w (Store Conditional) that tries to store and only succeeds if nothing changed since lr.w.



