2026-06-21 23:51

Tags: 

Functions are reusable blocks of code that can be called from other parts of a program.

```
fn main() {
    println!("Hello World!");
    main2();
}

fn main2() {
    println!("Hello Second World!");
}
```

Unlike C++, Rust allows functions to be called before their definition appears in the source file.

The Problem Rust is Solving: Rust parses the entire module before performing name resolution and type checking. This allows source files to be organized according to readability rather than declaration order.

Tradeoff: The compiler performs more work but the programmer gets more flexibility when organizing their code.


### Function Parameters

Function parameters require explicit type annotations.

```
fn enter_number(num: i32) {
    println!("Hello Second World!");
}

fn enter_number(num) {} // not allowed
```

This because Function signatures form part of a function's public interface. The compiler should be able to determine how a function is used without inspecting its implementation.


### Statements and Expressions

A statement is an instruction that performs an action but doesn't return a value.

```
let x = 5;
println!("Hello");
```

Statements mainly exist for their side effects.


An expression is an instruction that evaluates to a value.

```
3 + 5  
x * 2  
true
```


### Blocks are Expressions

Block scopes are expressions in Rust.

```
let y = {
    let x = 3;
    x + 1
};

println!("{}", y);
```

The value of the block is the value of its final expression so in this case it is 4.


### Semicolons and Value Suppression

Adding a semicolon makes the expression a statement. 

If we say something like `4 + 3` in Rust, the expression evaluates to 7 but if you do `4 + 3;` then the value is discarded.

That is the reason why earlier with the blocks we had this work

```
let y = {
    let x = 3;
    x + 1
};

println!("{}", y);
```

but this doesn't work

```
let y = {
    let x = 3;
    x + 1;
};
```

The value of `x + 1;` is discarded so the block statement doesn't know what the value is because of the semicolon that is suppressing the value.

This allows for more flexibility in my opinion with the tradeoff being that it can feel odd at first, especially since I'm coming from C++.


### Return Values

```
fn add(x: i32, y: i32) -> i32 {
    x + y
}
```

The final expression of the function becomes the return value. This expression is called the tail expression.

Of course just like before, adding a semicolon suppresses the value and generates a compiler error.

```
fn add(x: i32, y: i32) -> i32 { // expected i32 found ()
    x + y;
}
```

So function bodies and block expressions behave very similarly with both their values coming from their final expression.


### Returning Multiple Values

Functions can return multiple values using tuples.

```
fn add_sub(x:i32, y:i32) -> (i32, i32) {  
    (x + y, x - y)  
}  
  
fn main() {  
    let z = add_sub(10, 6);  
  
    println!("The values of the sum and difference are: {} and {}", z.0, z.1)  
}
```

This is pretty interesting to me. Compared to C++ that would do

`void add_sub(int x, int y, int& sum, int& diff);` 

Rust prefers to use values.


### Early Returns

The `return` keyword immediately exits a function.

```
fn add_sub(x: i32, y: i32) -> (i32, i32) {
    return (x + y, x - y);

    println!("hello");
}
```

The compiler warns that the statement after the return is unreachable.


### Questions for Later

- Why are if statements expressions in Rust?
- Why does a semicolon suppress a value?
- How does ownership interact with function arguments?



