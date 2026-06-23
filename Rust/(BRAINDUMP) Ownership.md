2026-06-22 23:27

Tags: 

Ownership seems to be one of the most important concepts in Rust from what I've heard.

There are a couple ways that languages handle memory. Some languages have a garbage collector regularly looks for no longer used memory as the program runs (Python, Java, Go). Some languages explicitly allocate/free memory (C, C++, Assembly).

Rust is different however because memory is managed through a set of rules that the compiler checks. If any rule is violated, the program won't compile. Ownership will not slow down the program.

Problem Rust is Solving: From what I've heard, garbage collected languages are easier for the programmer but they come with some performance overhead that can be bad for safety critical systems like autonomous vehicles. I don't exactly know what that overhead is because I didn't deep dive into it. I'd assume that it just takes up a considerable amount of extra memory space and the process of cleaning up old memory may take some extra time as well.


### The Stack and the Heap

Many languages don't have us think about the Stack and Heap too much, if even at all. I know when I was doing my first year of programming in Python I just thought the Stack and Heap were data structures to use. The concept of the Stack data structure does slightly apply to the Stack memory model.

So the Stack stores values in the order it gets them and removes the values in the opposite order (LIFO). Pushing is adding onto the Stack and popping is removing data off the Stack. All data stored on the Stack must have a known size and fixed size.

This is why the data of vectors is stored on the Heap instead of the Stack in C++. But the size, capacity, and pointer to the vector data have known sizes at compile-time so it can be stored on the stack.

When you put data on the Heap, you request some space. The memory allocator finds a big enough empty space in the Heap, marks it as being in use, and returns a pointer which has the location's address. 

Allocating space and Accessing data is harder on the heap. You have to follow the pointer wherever it needs to go. But all the data on the heap is unorganized wheras on the stack, data we need will likely be very close to each other.


### Ownership's Purpose and Rules

The main purpose of ownership is to manage heap data: to keep track of what code is using what data on the heap, minimizing the amount of duplicate data on the heap, and cleaning up unused data on the heap. 

1. Each value has an owner
2. There can only be one owner at a time
3. When the owner goes out of scope, the value will be dropped

I think that this is pretty interesting. Obviously the fact that ownership manages heap data is good because we want to make it the best possible way for a programmer to store and use heap data without causing a memory leak or heap corruption. 

Also the fact that there is only one owner makes it much easier for the programmer to know where the data is and who holds it. I tend to have some troubles once in a while knowing what data is holding what, which is why I really like smart pointers in C++.


### Variable Scope

```
fn main() {  
    // s is not valid here, it's not declared yet  
  
    let s = "hello world!"; // s is valid from this point forward  
  
    // using s here} // the scope is now over, so s is no longer valid
```

This is the same as C++ it seems, where the variable lifetime is tied to the scope that it's in. It looks like function scope here as well ends when the closing curly brace is identified.


### The `String` Type, Memory Allocation, `Drop` Function

```
fn main() {  
    // s is not valid  
  
    let mut s = String::from("hello"); // s is valid and on the heap  
  
    s.push_str(", world!");  
  
    println!("{}", s);  
} // s is not valid anymore, Rust calls drop
```

Drop reminds of the RAII pattern in C++ where once a resource is not being used the resource is automatically freed. I think that maybe ownership could be based on the RAII pattern in C++. I feel like I've heard that somewhere before.

Also seems like String objects in Rust are also stored on the heap just like in C++. I do wonder if there is something similar to C++'s Small String Optimization in Rust that stores strings that are short enough on the Stack.


### Move

```
fn main() {  
    let s1 = 10;  
    let s2 = s1;  
  
    println!("s2 = {}", s2); // prints s2  
  
    println!("s1 = {}", s1); // prints s1  
  
}
```

```
fn main() {  
    let s1 = "hello";  
    let s2 = s1;  
  
    println!("s2 = {}", s2); // prints s2  
  
    println!("s1 = {}", s1); // prints s1  
  
}
```

These two are perfectly fine. They use the Copy trait.

```
fn main() {  
    let s1 = String::from("hello");  
    let s2 = s1;  
  
    println!("s2 = {}", s2); // prints s2  
  
    println!("s1 = {}", s1); // compiler error!
  
}
```

error: move occurs because `s1` has type `String`, which does not implement the `Copy` trait

This is because when we do `s2 = s1;` for a type that is on the Heap, s1 no longer owns the String and s2 now owns the String. Ownership gets transferred. 

```
fn main() {  
    let s1 = String::from("hello");  
    let s2 = s1;  
  
    println!("s2 = {}", s2); // prints s2  
}
```

So this is perfectly fine.

This doesn't happen with elements on the Stack because Copy types are very trivial to duplicate. They don't have ownership ambiguity, they don't need resource cleanup, so Rust simply allows implicit copying for them.

```
fn main() {  
    let s1 = String::from("hello");  
    let s2 = s1.clone();  
  
    println!("s2 = {}", s2); // prints s2  
  
    println!("s1 = {}", s1); // prints s1  
  
}
```

This does work though, because `clone()` makes a deep copy of the heap data of the String, not just the Stack data. Deep copies are of course expensive.

I find the design choice interesting. In C++ deep copies are made simply just by passing by value in functions, doing `s2 = s1;`, and so on, but in Rust they make a deep copy possible by explicitly using a function. That seems better for programmers to be intentional with what they want to do.


### Copy Trait

The copy trait works for Stack-only data. Types like integers are stored on the stack so copies of the values are quick to make. So there is no need to call clone in this case, Rust sees having x and y both valid as trivial.

```
fn main() {  
    let x = 5;  
    let y = x;  
  
    println!("s2 = {}", y); // prints y  
  
    println!("s1 = {}", x); // prints x  
  
}
```

If a type has the Drop trait however (e.g String), you can't add the Copy trait.









