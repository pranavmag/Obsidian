2026-05-24 22:33

The IR essentially represents the middle-end of the compiler. The frontend with the Lexer and the Parser have so far allowed the compiler to understand the syntax of the code, which we produced an Abstract Syntax Tree (AST) from. But we don't have the meaning of any of these so far, just the syntax.

This is where the IR steps in and takes the 3D AST and walks it to unravel a 1D sequence of operations - a Linear IR. Every complex, nested expression is broken down into a straight line of simple, small-sized steps using virtual registers. 

The IR Taxonomy can be represented with three axes: Structural Organization, Level of Abstraction, and Naming Discipline.

