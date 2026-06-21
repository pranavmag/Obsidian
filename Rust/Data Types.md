2026-06-20 17:38

Tags: 

Scalar Types: Integers, Floating-point numbers, Booleans, Characters
Compound Types: Tuples, Arrays
Custom Types: Structs, Enums


### Integers

| Length  | Signed | Unsigned |
| ------- | ------ | -------- |
| 8-bit   | i8     | u8       |
| 16-bit  | i16    | u16      |
| 32-bit  | i32    | u32      |
| 64-bit  | i64    | u64      |
| 128-bit | i128   | u128     |
| arch    | isize  | usize    |
isize reminds me of size_t in C++. Also interesting they have 128-bit in here, i'm not actually sure if they have that in C++.


### Numeral System

Decimal Base-10, 1023
Hexadecimal Base-16, prefixed with 0x, 0xff
Octal Base-8, prefixed with 0o, 0o77
Binary Base-2, prefixed with 0b, 0b001
Byte (u8 only), ASCII characters, prefixed with b, b'A'

Honestly most of these are the same as C++.


### Floating Point

We have f64 (default) and f32. f64 is double-precision and f32 is single-precision.

```
let x = 3.14 // f64 default
let y:f32 = 3.0; // f32
```

I don't feel the need to write a section on the numeric operations. Written in the same way as C++ as most other languages I assume.


### Boolean

```
let x = true; // implicit declaration
let f:bool = false; // explicit declaration
```

`let b:bool;

You can't do this because bools cannot be uninitialized.


### Characters

4 bytes in size and used to represent a single unicode scalar value. This surprised me because in C++ it is usually 1 byte in size. What problem is Rust solving by making char Unicode instead of a byte? I looked into it and this likely allows for more character options to be able to use. For example common English characters take up one byte but complex symbols and emojis may take up to 3 or 4 bytes. That being said though, it could be wasting a lot more memory to store these very simple chars, because every single char even ones like 'a', '1', and 'Z' are all 4 bytes. Is there a way to possibly split this up?

```
let c = 'a';
let x = '1';

// You can also iterate through each character in a string
for char in "yes no maybe".chars() {  
    println!("{}", char);  
}
```


### Tuples

Tuples are a way to group different values together.

```
let tup: (i32, f64, char) = (200, 3.14, 'A');

let (x, y, z) = tup;  
  
println!("The value of y is {}", y);
```

As you can see we can put the individual values of tuples into individual variables. This is known as destructuring.

However we cannot change the structure of the tuple, even if we make it mutable.

```
let mut tup: (i32, f64, char) = (200, 3.14, 'A');  
tup = (200, 3.14, 'A', true);  // Does not work!
let (x, y, z) = tup;  
  
println!("The value of y is {}", y);
```

This compiler will generate a warning telling you that it expected the three specific types and you ended up giving it 4 types. 

Even if you do something like this the compiler will complain that it expected a char.

```
let mut tup: (i32, f64, char) = (200, 3.14, 'A');  
tup = (200, 3.14, 2.0);  // Compiler Error!
let (x, y, z) = tup;
```

If you use shadowing then of course you can restructure it but that's only because shadowing creates an entirely new binding.

```
let tup: (i32, f64, char) = (200, 3.14, 'A');  
let tup = (200, 3.14, 'A', true);  
let (x, y, z, a) = tup;  // The older tup binding is hidden so you have to update to the new binding for tup
  
println!("The value of y is {}", y);
```

We can also access single tuple elements by index.

```
let tup: (i32, f64, char) = (200, 3.14, 'A');  
  
let two_hundo = tup.0;  
let pi = tup.1;  
let ay = tup.2;  
  
println!("{}, {}, {}", two_hundo, pi, ay);
```

I think tuples would be very useful for fields of data. They kind of remind me of structs now that I think about it. Structs aggregate a bunch of data into a structure. I used tuples in Python and now that I used C++ I used structs but I never thought of the comparison between the two until now. But I guess the main difference is that structs have named fields whereas tuples don't have named fields. So my guess is that structs are more useful if many different objects with different values need to be created using that same field of data and separated with aliases, and tuples are better used if that data needing to be used is consistent but is used a lot.


### Arrays

An array is a collection of elements of the same type with a fixed-length. The size cannot be altered after declaration, so it is not a dynamic array.

```
let array = [1, 2, 3, 4, 5]; // array of 5 elements

let first = array[0] // access element at index 0

for element in arr.iter() {
	println!("{}", element);
}
```

my guess: I assume that the iter() member function is something similar to C++ where it is a pointer that holds the memory address of the first element and as we loop through it the pointer moves through the contiguous chunk of memory.

actual answer: A Rust iterator is more like an object that knows how to produce values one at a time, not necessarily a pointer.


### Structs

```
struct Person {  
    name: String,  
    age: u8,  
}  
  
fn main() {  
    let person = Person {  
        name: String::from("Nav"),  
        age: 20,  
    };  
  
    println!("The person's name is {} and the person's age is {}", person.name, person.age)  
}
```


### Enums

This allows us to create enumerations.

```
enum Animal {  
    DOG,  
    CAT,  
    FISH,  
}  
  
fn main() {  
    let animal = Animal::DOG;  
  
    match animal {  
        Animal::DOG => println!("woof!"),  
        Animal::CAT => println!("meow!"),  
        Animal::FISH => println!("glug!"),  
    }  
}
```

This reminds me a lot of how C++ does it with the scope identifier telling us that we need to look at the Animal namespace and look for the DOG enumeration and then to do a task based on that. The match statement reminds me a lot of the switch-case statement in C++.

### Questions for Later

- Why do usize/isize exist?
- Why does Rust prefer iterators so heavily?
- When should I use a tuple versus a struct?
- How are Rust enums different from C++ enums?