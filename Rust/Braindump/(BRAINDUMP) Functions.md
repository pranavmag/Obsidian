2026-06-21 02:36

Tags: 

A function is a reusable piece of sequential code. 

```
fn main() {  
    println!("Hello World!");  
    main2();  
}  
  
fn main2() {  
    println!("Hello Second World!");  
}
```

This is allowed. This means that the order of functions does not matter. This is different from C++ where the functions have to be declared before the functions that calls it or have a forward declaration telling the compiler that the function exists at some point. 

This provides much more flexibility for the programmer.

What other problems is Rust solving here? I do think that another reason it could be helpful is that the compiler parses the entire module before performing name resolution and type checking. Source files can be organized for readability rather than dependency order.

Tradeoff: Compilation becomes slightly more sophisticated. I do wonder if that's why Rust compile times tend to take longer (or so I've heard).

```
fn enter_number(num) {  // Compiler Error!
    println!("Hello Second World!");  
}  
  
fn main() {  
    println!("Hello World!");  
  
    let x = 3;  
    enter_number(x);  
}
```

The compiler will ask us to specify the type.

```
fn enter_number(num: i32) {
    println!("Hello Second World!");  
}  
  
fn main() {  
    println!("Hello World!");  
  
    let x = 3;  
    enter_number(x);  
}
```

Why can Rust infer local variable types but not function parameter types?

It seems that the answer is because function signatures form part of the public interface so the caller should not have to inspect the function body to know its parameter types.


### Statements and Expressions

A statement is an instruction that performs an action but doesn't return a value.
An expression is an instruction that evaluates to a value.

```
let x = 3 + 5; // statement
// but the 3 + 5 part is an expression
```

```
let x = (let y = 42); // compiler error!


let y = {
	let x = 3;
	x + 1
};

println!("The value of y is: {}", y) // 4
```

A block scope like this is considered an expression. 

```
let y = {
	let x = 3;
	x + 1; // If we put semicolon then compiler error
};

println!("The value of y is: {}", y) // 4
```

The last expression of a block statement is the value that is being returned. The semicolon is not a part of the expression, it is what turns that expression into a statement. 

The semicolon suppresses the value. For example if we have the expression statement 4 + 3;
The value that is evaluated is 7 but the semicolon just leaves that 7 there. But in the case of the block statement we have the x + 1 not terminated with a semicolon so when the block ends that value of x + 1 is still there and the compiler knows that the block evaluates to 4.


### Return Values

We can see the same thing we just recently mentioned with function returns.

```
fn add(x:i32, y:i32) -> i32 {  
    x + y  
}  
  
fn main() {  
    let z = add(3, 4);  
  
    println!("The value of z is: {}", z) // 4  
}
```

It seems that add() has the expression x + y but since it doesn't have a semicolon the value still exists for main() to be able to use.

Adding a semicolon gives the error, "implicitly returns `()` as its body has no tail or `return` expression" and even shows that the function expects a return value of i32 not (). As we said earlier, the semicolon is suppressing the value. This is a very interesting design choice imo.

I'm used to the semicolon just being a statement terminator but not having anything to do with the values of expressions. But maybe there is something similar in other languages but I just never had to give it a thought.

```
fn add_sub(x:i32, y:i32) -> (i32, i32) {  
    (x + y, x - y)  
}  
  
fn main() {  
    let z = add_sub(10, 6);  
  
    println!("The values of the sum and difference are: {} and {}", z.0, z.1)  
}
```

We can also use tuples to return multiple types. As you can see here I made the function expect to return a tuple of two values.

`println!("The values of the sum and difference are: {:?}", z)` 

The last line could also be this to just print out the values of the tuple in succession.

So this is very interesting to me. We can return multiple values by using a tuple. I like that. I'm used to the C++ way where we use references or pointers.

`void add_sub(int x, int y, int& sum, int& diff);` 

Seems like Rust prefers returning values? I'm interested in the memory and under the hood.

### Early Return

We can use the `return` keyword to stop the flow of the function and return at any point.

```
fn add_sub(x:i32, y:i32) -> (i32, i32) {  
    return (x + y, x - y);  
    println!("hello numbers"); // compiler warning: unreachable statement
}
```

As you can see, we can return out early but any code after it will not be executed. So using an if statement or something will help not generate that compiler warning because now it can possibly reach the expression that was previously said to be "unreachable".

```
fn add_sub(x:i32, y:i32) -> (i32, i32) {  
    if x == y {  
        return (x + y, x - y);  
  
    }  
  
    (0, 0)  
}
```







