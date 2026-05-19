2026-05-16 18:34

### Hybrid Architecture

Instead of a tree-walk interpreter we're building a compiler. The frontend produces an AST and the backend walks that tree to do register allocation and emit machine code. The frontend itself is split into two strategies depending on what it's parsing:

Statements and Control Flow (if, while, return, variable declarations) -> Recursive Descent Parsing
These have clear ambiguous prefix keywords that are easy to parse top-down.

Expressions (a + b * (c-d)) -> Pratt Parsing
This handles complex precedence rules with a clean single-function design. 

The output of both -> AST nodes. Neither emits machine code directly. They build a tree that the backend traverses later.

The reason to build the tree first is that the backend needs to see the whole expression before it can make decisions. It needs to analyze variable lifespans, allocate registers efficiently and then emit your target instructions (add, mul, jalr, etc.) in the correct order for the five-stage pipeline. You can't do that while parsing, so we need to make the full picture first.

The Full Pipeline:

Source string 

     |
     v

  Scanner  →  token stream

     |
     v

  Parser

  ├── Recursive Descent  →  statement AST nodes (if, while, return...)

  └── Pratt Parser       →  expression AST nodes (Binary, Unary, Literal...)

     |
     v

  AST (the full tree)

     |
     v

  Backend traversal

  ├── Liveness analysis / register allocation

  └── Code generation  →  add, mul, jalr, ...


### Representing Code - AST Nodes

Context-Free Grammars (CFGs): The Lexer used a regular grammar. Regular grammars work for flat token streams but can't handle expressions that nest arbitrarily. A CFG handles this. It has:

Terminals - Actual tokens from the Lexer and End Points. They don't generate further
Nonterminals - Named references to other rules
Productions - The rules. Each has a name (head) and a body. Multiple productions for the same head are separated by |

Notation summary:

|                                                                                                                                |
| ------------------------------------------------------------------------------------------------------------------------------ |
| \|     means "or"<br><br>*     zero or more<br><br>+     one or more<br><br>?     optional<br><br>( )   grouping within a rule |

AST Node Types:

Expression Nodes