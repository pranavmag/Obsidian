2026-06-27 19:20

Tags: 

Slices in Rust are a dynamically-sized view into a contiguous sequence of elements in a collection. It borrows part (or all) of another collection without taking ownership. They act a view into the original collection, and do not store any data themselves. A slice consists of a pointer to the first element and a length. Slices are used to give a part of a collection to a function or to iterate over a part of a collection.

When I see the phrase "view into the original collection" it kind of reminds me of `std::string_view` in C++ where it gives us a view of a string and doesn't actually have the means to store its own data. It seems like both provide a non-owning view into existing data rather than allocating new storage.

Slice Syntax: `&[T]`
T is the type of the elements in the collection.
& is a reference to the collection.
So `&[T]` is a borrowed slice of elements of type `T1`

```
fn main() {  
    let arr: [char; 5] = ['a', 'b', 'c', 'd', 'e'];  
    let slice: &[char] = &arr[1..3];  
    println!("{:?}", slice);  // ['b', 'c']
}
```

So it looks like it takes a slice of the array from index 1 to 3 (exclusive). `&a[start..end]` is what it follows.

One thing I like here is the array syntax. We use `[T; N]` here which is pretty straightforward, `T` for type and `N` for the amount of elements.

```
fn main() {  
    let s = String::from("Nav");  
  
    let slice = &s[0..3];  
    println!("{}", slice); // Nav
    let slice = &s[..3];  
    println!("{}", slice); // Nav
}
```

You can have range shortcuts like this.


### Why Slices?

`fn print(s: &String)`

`fn print(s: &str)`

`&str` can accept String, &String, string literals, and existing string slices

`&String` can only accept String

It also seems that a slice is really just two things: pointer and length

`let slice = &arr[1..4];`

```
pointer --> arr[1]
length = 3
```


``



