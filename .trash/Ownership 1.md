2026-06-23 21:00

Tags: 

Ownership is one of Rust's most important concepts. It's Rust's solution to memory and resource management without needing a garbage collector.

Lots of other languages use different styles of memory management.

Garbage collected languages like Python, Java, and Go automatically find and free unused memory during program execution. Manual memory management languages like C, C++, and Assembly have the programmer explicitly manage memory or use abstractions such as RAII and smart pointers.

Rust does it differently by having a set of compile-time rules enforce ownership, giving a compiler error if a rule is broken. 

Problem Rust is solving: Preventing common memory bugs like use after free, double free, dangling pointers, most memory leaks. Also no garbage collector overhead at runtime.

Ownership determines who owns a resource and when that resource should be cleaned up.


### The Stack

The stack stores data in a Last-in, First-out (LIFO) order. This allows for fast allocation/deallocation and really good cache locality. The data on the stack must have a known, fixed size at compile time. Primitive types like integers, chars, bools, are stored on the stack.


### The Heap

The heap is used for data that has a size that may not be known at compile time or may change during execution. Allocation on the heap requires requesting memory from the allocator and the access of this data usually involves using pointers to follow the data. The memory may be scattered throughout the address space which makes heap allocation more expensive because cache locality is reduced, finding free memory may be harder, and pointer indirection could be hard.

Types like `String` and `Vec<T>` have their data stored on the heap and their metadata stored on the stack.


### Ownership Rules

1. Each value has an owner
2. There can only be one owner at a time
3. When the owner goes out of scope, the value is dropped


### Variable Scope

```
fn main() {
    let s = "hello";

    println!("{}", s);
}
```

When the scope ends, the variable is no longer valid. This behavior is similar to local variable lifetimes in C++.


### String and Heap Allocation

```
let s = String::from("hello");
```

allocates memory on the heap and owns that memory.

Because the data is owned, it can be modified:

```
let mut s = String::from("hello");
s.push_str(", world!");
```


### Drop and Resource Cleanup

When an owner goes out of scope, Rust automatically calls `drop`.

```
fn main() {
    let s = String::from("hello");
}
```

At the end of the scope, Rust automatically frees the resources owned by `s`.

This behavior is similar to RAII in C++.

C++:

```
std::string s = "hello";
```

The destructor runs automatically when the object leaves scope.

Rust follows the same idea but enforces ownership rules through the compiler.


### Moves

Ownership can be transferred between variables.

```
let s1 = String::from("hello");
let s2 = s1;
```

After this assignment:

- `s2` owns the String
- `s1` is no longer valid

```
println!("{}", s1); // compiler error
```

This is called a move.

The ownership is moved here. The important change is not the pointer itself but who is responsible for the resource. After the move, only one owner exists. This prevents accidental double-frees.


### Cloning

If two independent copies are needed, a deep copy must be performed explicitly.

```
let s1 = String::from("hello");
let s2 = s1.clone();
```

- `s1` owns one String
- `s2` owns another String

Both variables remain valid and can be used. Rust makes expensive operations visible. A deep copy may require allocating memory and copying heap data. Rust makes the cost explicit to the programmer.

This is the opposite of C++ who allows even just assigning a variable to a different variable can be a deep copy. It looks cheap but it could actually be expensive.


### Copy Types

Some types implement the `Copy` trait.

Examples:

```
i32
u64
bool
char
```

These types are inexpensive to duplicate.

```
let x = 5;
let y = x;

println!("{}", x);
println!("{}", y);
```

This works because the value is copied rather than moved.

Types that implement `Drop` cannot implement `Copy`. If such types could also be copied implicitly, multiple objects might attempt to free the same resource. Rust prevents this by disallowing `Copy` on types that require cleanup.


### My Thoughts

Ownership feels like RAII enforced at the language level. Rather than relying on programmer discipline, the Rust compiler verifies ownership and cleanup rules before the program can run.



