2026-06-22 00:45

Tags: 

### If Statements

If statements are really cool in Rust.

```
fn main() {  
    let condition = true;  
  
    let number = if condition {  
        5  
    } else {  
        6  
    };  
  
    println!("{}", number);  // 5
}
```

This is possible because if statements are actually expressions in Rust. So they can work exactly like how those block expressions work. This is already very flexible compared to other languages. In C++ I'd probably use a ternary expression to get this but I definetely like this much better. You can write much more code in the if and else blocks and have the value of the end expression returned. Certainly can't do that with ternary expressions lol.

Everything else works the same way as other languages: conditional operators, equality operators, nested if-else statements, else if, modulus, etc.


### Match Construct

```
fn main() {  
    enum Coin {  
        PENNY,  
        NICKEL,  
        DIME,  
        QUARTER,  
    }  
  
    fn value_in_cents(coin : Coin) -> u8 {  
        match coin {  
            Coin::PENNY => 1,  
            Coin::NICKEL => 5,  
            Coin::DIME => 10,  
            Coin::QUARTER => 25,  
        }  
    }  
  
    let coin = Coin::PENNY;  
  
    println!("Value of coin: {}", value_in_cents(coin));  
}
```

As we know from the data types page we can use enumerations to define aliases and use a match statement to identify which one is being used.

The great thing is that if I forget to implement `COIN::QUARTER` in the match statement the compiler generates an error to tell me about it.

As I said before, it reminds me of a switch case statement used with enums in C++. I'm excited to see in what other ways enums could possibly be used.


### Loops

The infinite loop uses `loop`

```
fn main() {  
    loop {  
        print!("Hello World!");  
    }  
}
```

`loop` is also an expression so we can do something like

```
fn main() {  
    let mut counter = 0;  
  
    let result = loop {  
        counter += 1;  
  
        if counter == 10 {  
            break counter * 2;  
        }  
    };  
  
    println!("The result is {}", result);  // 20
}
```

The most interesting thing I see here is that we can say `break counter * 2`. That seems very interesting considering that break immediately takes you out of the loop in most languages and doesn't return anything.


A while loops executes as long as a condition is met.


### For Loops

There aren't common for statements in Rust unlike other languages.

```
fn main() { 
  // loops through each element
    let a = [1, 2, 3, 4, 5];  
  
    for element in a.iter() {  
        println!("The value is {}", element);  
    }  
  
  // loops through each char
    let s = "hello world";  
  
    for c in s.chars() {  
        println!("The value is {}", c);  
    }  
  
  // For Loop from Range 1-4(exclusive)
    for numer in 1..4 {  
        println!("The value is {}", numer);  
    }  
    
  // For Loop from Range 1-4(inclusive)
    for numer in 1..=4 {  
        println!("The value is {}", numer);  
    }  
}
```


### Design Observations

Rust Feature: `if` is an expression
Problem Rust is Solving: Allows control flow to produce values and compose naturally with other expressions
Tradeoff: Can initially feel unusual to programmers coming from other languages

Rust Feature: `loop` is an expression and `break` can return a value
Problem Rust is Solving: Allows loops to directly produce results rather than requiring mutable variables outside the loop
Tradeoff: Adds another concept that must be understood when learning control flow but very trivial time to learn in my opinion

Rust Feature: `match` requires exhaustive handling of all cases.
Problem Rust is Solving: Prevents forgotten cases and catches many logic errors at compile time.
Tradeoff: Requires additional code when all cases are not immediately needed.

Thought: Rust keeps treating control-flow constructs as value-producing expressions rather than purely execution-flow statements. it seems to be their unique design choice.


### Questions for Later

- Why does Rust prefer iterator-based loops over traditional C-style loops?
- How does `break value` work under the hood?











