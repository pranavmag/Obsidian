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

### AST Node Types

#### Expression Nodes

For expression nodes you can have many different types of expressions, so you could set up a base class that has multiple derived classes for each type of expression. This is because expressions can be nested infinitely like (a +b) * c, and the parser needs a polymorphic way to say, "This child could be any type of expression." By having everything inherit from the base class then a BinaryOpNode can hold a Literal node value on it's left and a function call node on its right without actually needing to know the exact types upfront. For these we use Pratt Parsing

For expression nodes you'll have a couple of types like Binary Op Nodes, Unary Op Nodes, Literal Nodes, Identifier Nodes, and Function Call Nodes.

#### Statement Nodes

For statement nodes it's pretty much the same idea but we use recursive descent parsing. We will again have a base class that has a bunch of derived classes for each type of statement.

For statement nodes you'll have a couple of types like Variable Declaration Statements, Return Statements, If Statements, While Loop Statements, Block Statements, and Expression Statements.

These would be stored in a file like ast.h


### Visitor Pattern

The visitor pattern makes it so that you can define behavior separately from the AST Nodes. Compilers usually perform many passes over an AST so if you didn't have visitors then you'd have to implement tons of methods inside each AST node struct.

With visitors, each compiler pass becomes its own class. The problem it is really solving is that we have a tree of polymorphic node types but we want different behavior depending on the actual runtime types.

Just implement one visitor method per concrete type and then implement accept() functions for each. 

The Visitor Pattern allows for double dispatch. Dispatch essentially just means "which function gets called?" Virtual functions only dispatch on one runtime type, not two so the visitor pattern is a workaround to simulate two dynamic types deciding which function runs. 

Single dispatch (normal virtual functions):

Expr* e = new BinaryOpNode();
e->print();

C++ chooses print() based on the runtime type of e -> BinaryOpNode. So there is one dynamic decision (receiver object).

Double Dispatch allows a function to be called based on BOTH the node type (BinaryOp, Literal, etc.) and the operation (Print, Evaluate, Optimize, etc.). Single dispatch is not enough in this case because if you need to add a new operation then you need to modify every single node.

the accept() functions for each allow us to perform visitor.visit(expr); because we can't actually use that because C++ cannot pick the exact child nodes it needs. So all the accept function does is Re-expose the concrete type to the visitor in a proper way. 

Your visitor implementation should have:

Base Visitor Interface - One method per concrete type, usually pure virtual
Node Classes have accept(visitor) - Always takes visitor as a parameter
accept() forwards *this - v.VisitX(*this);
No virtual visit() inside nodes themselves - Nodes do not implement behavior, they delegate it out

This can be implemented in a visitor.h file.


### Parser

Now we get to the parser itself. It's entire job is to take the array of tokens that the lexer generated and construct the AST.

This is where you'll actually implement the functions we made earlier so set up the class to take in that vector of tokens and initialize the class member functions. You should also be adding some helper functions which we will talk about:

match(types): This looks at the current token. if it matches the expected type (for example if the next token is an = sign), then it automatically consumes it and advances the cursor. Use std::initializer list here to take a brace-enclosed list of TokenType values.

consume(type, message): This is the strict enforcer of your grammar. If you are parsing a while loop, you must have a closing parenthesis. consume(TokenType::RIGHT_PAREN, "Expect ')'") demands that the token is there. If it isn't, the user wrote bad C code, and consume will throw an error, preventing the compiler from generating broken RISC-V instructions.

peek() and previous(): These let the parser look around without committing. peek() looks at the token at current_ without advancing, which is crucial for the Pratt parser's while (bindingPower(peek()) > minBP) loop to decide if it should keep grabbing operators or stop.








