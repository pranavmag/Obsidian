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


### Basic Blocks

The Basic Block is a list of sequential instructions that are guaranteed to execute together. The fundamental rule of a Basic Block is: Enter at the top, exit at the bottom. 

You can't jump to the middle of a Basic Block from somewhere else, you can only enter in through the first instruction. There are no BEQ or JAL instructions hidden in the middle of the block. A branch or jump is only ever allowed to be the very last instruction in the block.


### Control Flow Graph (CFG)

While the AST represents code as a nested tree, our RISC-V CPU does not understand nesting, they only understand jumping to different memory addresses. The Control Flow Graph (CFG) is the data structure that replaces the AST in the Middle-End.

The Structure: A CFG is a web of Basic Blocks. Each block contains a vector representing straight line code

Dynamic Spawning: As the IR Generator walks down an if statement or while loop, it dynamically creates new Basic Block objects on the fly, emits conditional branch Quads, and wires the block pointers together before recursively exploring the inner blocks.


### AST to Linear IR via Post-Order Traversal

We can use a post-order traversal to flatten the Pratt-parsed expressions into a 1D sequence.

The IR Generator which acts as our ExprVisitor has to evaluate the children before it evaluates the parent operator.

Every single visit method for an expression must return an int representing the virtual register id that holds the result of that node.

When visiting a + b * c, the + node asks its left child (a) for its vreg.
The + node asks its right child (*) for its vreg.
The * node is forced to evaluate b and c, emits a Multiply Quad, and hands its resulting vreg back up to +.
Finally, the + node emits an Add Quad. The recursive unwinding naturally generates the instructions in the exact correct machine execution order.

Simplified Symbol Table: We aren't allowing nested blocks at least for the time being so we can just have one single flat map for the whole function instead of a struct with parent pointers.


### Middle-End Translator's Workflow

With only one flat map, the Middle-End Translator's job of walking the AST becomes straightforward.

Simplified Design: Because we altered the Parser to **strictly prohibit arbitrary nested blocks**, all local variables exist at the same function-level scope.
    
Flat Map: We do not need a complex "Sheaf of Tables" or linked list of Environments. We only need a single `std::unordered_map<std::string, int>` per function.

Lifecycle: Entering a function -> `.clear()` the map.
    
visitVarDecl -> Check if it exists (error if true), allocate a new vreg, and add to the map.
        
visitIdentifier -> Look up the variable name in the map and return its assigned vreg ID.
