2026-05-24 22:33

The IR essentially represents the middle-end of the compiler. The frontend with the Lexer and the Parser have so far allowed the compiler to understand the syntax of the code, which we produced an Abstract Syntax Tree (AST) from. But we don't have the meaning of any of these so far, just the syntax.

This is where the IR steps in and takes the 3D AST and walks it to unravel a 1D sequence of operations - a Linear IR. Every complex, nested expression is broken down into a straight line of simple, small-sized steps using virtual registers. 

The IR Taxonomy can be represented with three axes: Structural Organization, Level of Abstraction, and Naming Discipline.

My choice for Structural Organization will be a Linear IR which represents the 3D AST as a 1D linear sequence of instructions. The alternatives are Graphical and Hybrid.

The Level of Abstraction I am choosing is Low-Level which would be near the machine code for RISC-V. The alternatives are source code level or mid level.

My choice for Naming Discipline is virtual registers (SSA optional), choosing to use virtual registers to have more registers to work with and then map to real physical registers down the line. The alternatives are source names, stack model, and memory-to-memory.


### Three Address Code

Three Address Code is very similar to how RISC-V assembly looks. 

t3 <- t1 + t2   // Three Address Code
add x3, x1, x2  // RISC-V Assembly

Each t name is a virtual register, acting as a symbolic name with no physical register assigned to it yet. Register allocation (a later compiler pass) maps these onto the real 32 RISC-V registers.


### Quads

A Quad represents the values of the TAC, the two operands, the operator, and the destination. The text says that most compilers use an array of Quads within each CFG node (basic block). A Basic Block is essentially just a sequence of straight-line code. Execution enters at the top and leaves at the bottom with no branching in the middle.

The trade off with the array of quads was that it is fixed-size and that resizing requires copying. Nowadays we have vectors so I'm pretty sure I could use a vector so that those trade offs aren't there any more. So in that case, I wouldn't have to use a linked list of quads or an array of pointers to quads. These were other approaches that were used, not seen so much nowadays.

This would likely be easier than having a linked list of quads or an array of pointers because we don't have to jump all over RAM to read the next instruction. 


### Virtual Registers

Virtual Registers are registers that are separate from the physical registers. They are pseudo registers that have a name for a value. There can be an infinite amount of virtual registers which makes it more flexible for us right now. Later when we do register allocation we will actually map the virtual registers to the 32 physical registers of RISC-V.

Every local variable that we have will become an alias for a virtual register ID. We don't deal with memory right now, so we don't need to worry about LOAD/STORE at the moment.


