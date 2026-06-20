2026-06-20 00:17

Tags: 

### Variable Bindings with let
Variables are created using the `let` keyword and Rust infers the type automatically when it can. The more common way I tend to see it expressed is that let creates a binding between a name and a value.

`let x = 3;` is binding the name x to the value 3


### Immutability by Default

Variables created with `let` are immutable unless explicitly marked as mutable.

```
let x = 3;  
x = 4; // Error

let mut x = 3;
x = 4; // OK
```

Rust encourages immutable data by default because immutable values are generally easier to reason about and less prone to bugs.

Whereas in C++ where immutability is an opt-in, Rust makes immutability the default.


### Mutability Changes Values, Not Types

```
let mut x = 3;
x = 4; // OK

let mut x = 3;
x = "hello"; // Error
```

Once a binding has a type, that type remains fixed. This is because the compiler needs to know exactly what type a variable has throughout its lifetime. Allowing variables to freely change type would complicate type checking and program reasoning. 


### Shadowing

Rust allows a new binding to reuse an existing name

```
let x = 3;
let x = 4;
```

The second binding shadows the first, the original binding still exists but the new binding hides it. Shadowing does not modify the original variable. It creates a completely new binding.


### Shadowing Can Change Types

Since shadowing creates a new binding, that new binding can have a different type.

```
let x = 3;
let x = "hello";
```

Using shadowing preserves immutability while allowing the value to evolve through processing steps.

The problem Rust is solving: Allows transformations without requiring mutable state

Tradeoff: The same name may refer to different bindings, which can initially be confusing.

My Thoughts: This feels similar to SSA (Static-Single Assignment) form in compiler design, where each new value is conceptually a new variable.


### Constants

Rust supports compile-time constants using const

`const MAX_POINTS: i32 = 100_000;`

Unlike `let`, constants require an explicit type annotation. This is because constants are evaluated at compile time and become part of the program itself, so the compiler requires a fully specified type.


### let vs const

`let`: Runtime binding, Type inference supported, May be mutable with mut, Exists when program runs

`const`: Compile-time constant, Type annotation required, Always immutable, Embedded into the compiled program

Rust `const` is actually more closer to C++ `constexpr` rather than C++ `const` it seems like.


### Numeric Separators

Rust allows underscores in numeric literals for readability

```
let x = 100_000;

let x = 100000; // The same thing
```

### Formatting with println!

Values are inserted into strings using formatting placeholders.

`println!("The value is {}", x);`

The `{}` acts as a placeholder for the provided argument.

```
let x = 5;
let y = 10;

println!("x = {}, y = {}", x, y);
```

Arguments are matched to placeholders in order.

Observation: This resembles string interpolation, but Rust uses explicit formatting placeholders rather than embedding variable names directly inside the string.


### Questions for Later

- How are constants represented in the compiled binary?
- How does shadowing interact with ownership?
- Is the SSA mental model for shadowing actually accurate?
- When should I prefer shadowing over mutability?