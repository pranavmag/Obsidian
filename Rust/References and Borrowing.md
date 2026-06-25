2026-06-23 21:27

Tags: 

```
fn main() {  
    let x = 3;  
    call_int(x);  
    println!("The value of x is {}", x);  
  
    let s = String::from("Hello");  
    call_string(s);  
}  
  
fn call_int(i : i32) {  
    println!("The value of i is {}", i);  
}  
  
fn call_string(s : String) {  
    println!("The value of s is {}", s);  
}
```

This works and outputs perfectly fine, but when we add 

```
fn main() {  
    let x = 3;  
    call_int(x);  
    println!("The value of x is {}", x);  
  
    let s = String::from("Hello");  
    call_string(s);  
    println!("The value of s is {}", s); // added here
}  
  
fn call_int(i : i32) {  
    println!("The value of i is {}", i);  
}  
  
fn call_string(s : String) {  
    println!("The value of s is {}", s);  
}
```

then it does not compile. I think it is because String does not have a `Copy` trait so it won't allow both String variables to stay valid. The ownership of the value of `s` from our main function is is now moved to `s` in the `call_string` function. 


### Return Values and Scope

```
fn main() {  
    let s1 = give_ownership();  
    println!("s1: {}", s1);  
  
    let s2 = String::from("Hello from main");  
  
    let s3 = take_and_give_ownership(s2);  
    println!("s3: {}", s3);  
}  
  
fn give_ownership() -> String {  
    let some_string = String::from("Hello from give_ownership");  
    some_string  
}  
  
fn take_and_give_ownership(some_string : String) -> String {  
    some_string  
}
```

So from what I can see here, s1 is made in the `give_ownership` function and it gives the ownership back to s1 in main(). s2 is made in main and is given ownership to the `take_and_give_ownership` function but since that function ends up returning that same argument parameter back, now s3 has it at the end so the output is

```
s1: Hello from give_ownership
s3: Hello from main
```


### Borrowing

```
fn main() {  
    let s1 = String::from("hello");  
  
    let len = calculate_length(&s1);  
  
    println!("The length of '{}' is {}", s1, len);  
}  
  
fn calculate_length(s : &String) -> usize {  
    let length = s.len();  
    length  
}
```

We can pass the String by reference here. The action of creating a reference is called borrowing.

```
fn main() {  
    let s = String::from("hello");  
    change_borrowed_value(&s);  
}  
fn change_borrowed_value(s : &String) {  
    s.push_str(" world");  // compier error
}
```

We can't do this it seems like though. An `&` reference can't be borrowed as mutable. Only if we do `&mut` then we can borrow it as mutable. 

```
fn main() {  
    let mut s = String::from("hello");  
    change_borrowed_value(&mut s);  
    println!("{}", s);  
}  
fn change_borrowed_value(s : &mut String) {  
    s.push_str(" world");  
}
```

The variable is mutable, and we are passing in a mutable reference.

I think this is a very interesting language design choice. Whereas in other languages like C++ a programmer could accidentally change the value of a reference without meaning to, Rust yet again makes things like this need to be explicitly changed.

It's like Rust expects the programmer to make these mistakes so they have lots of guardrails in place whereas C++ is more like "I trust you'll ensure you're right".


### Dangling References

```
fn main() {  
    let ref_value = dangle();  
}  
  
fn dangle() -> &String{  
    let s = String::from("hello");  
    &s  
}
```

This does not work. We create `s` inside the `dangle()` function and we return the `&s` but the problem is that this value does not exist because as we know the heap allocated String will be dropped at the end of the function. So this would be a dangling reference.

This is amazing that Rust actually tells us that we have a dangling reference. 


### References Rules

1. At any given time, you can have either one mutable reference or any number of immutable references.
2. References must always be valid.

From what I've seen so far, I can understand why Rust is regarded as having strong memory-safety guarantees while still compiling to efficient native code.
