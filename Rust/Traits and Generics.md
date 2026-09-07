2026-09-06 19:30

Tags: 
### Traits

Traits define behavior and generics let us write code that works with many types. Trait bounds connect the two.

```
trait Speak {
    fn speak(&self); // defines default behavior
}

struct Dog;
struct Cat;

impl Speak for Dog { // different types can now use that behavior their own way
    fn speak(&self) {
        println!("Woof!");
    }
}

impl Speak for Cat {
    fn speak(&self) {
        println!("Meow!");
    }
}
```

If we didn't implement any behavior for `dog` then it would get the default implementation.

```
struct Dog;
struct Cat;

impl Speak for Dog {} // default behavior from trait

impl Speak for Cat {
    fn speak(&self) {
        println!("Meow!");
    }
}
```

### Generics

In the case of generics we have functions that can work for many types. T is a generic type parameter and it essentially means that this function works for some type T.

```
fn print_something<T>(value: T) {
    // ...
}
```

But the problem here is that generics alone don't tell us what T can do. Rust doesn't know what T can be printed so we need to tell Rust that T can be any type as long as it implements this certain behavior. So we can use trait bounds.

```
fn print_something<T: std::fmt::Display>(value: T) {
    println!("{}", value);
}
```

```
fn foo<T: Speak + Clone>(value: T) { // multiple trait bounds
}
```

So here we are saying that T must implement Display, therefore {} formatting is available.

```
Generic
   +
Trait Bound
   ↓
Generic code that can rely on specific behavior
```

`T: Trait` is a constraint, for example if we have `T: Speak` that means T must implement Speak.

```
fn make_speak<T: Speak>(animal: T) { 
	// give me any type T, provided that T implements speak
    animal.speak();
}
```

### Monomorphization

Generics use monomorphization meaning that Rust knows the concrete types and during compilation it can effectively generate specialized versions. There doesn't have to be a lookup for what type T is during runtime, so generics can be very fast.

The tradeoff is that more generic combinations can mean more compilation work, potentially more generated machine code, and potentially larger binaries. So generics aren't free from a compliation perspective.

```
fn foo<T>(value: T)
where
    T: Speak + Clone,
{
}

fn process<T, U>(value: T, other: U)
where
    T: Speak + Clone,
    U: Display + Debug,
{
}
```

The `where` clause can also be used and it is often easier to read when we have several constraints like in the second version.

Generics for structs also exist too.

```
struct Container<T> {
    value: T,
}

fn main() {
	let a = Container { value: 10 };
	let b = Container { value: "hello" };
}
```

### Blanket Implementations

```
trait Speak {
    fn speak(&self);
}

trait Describe {
    fn describe(&self);
}

impl<T: Speak> Describe for T {
    fn describe(&self) {
        self.speak();
    }
}
```

We can read `impl<T: Speak> Describe for T` as "For every type T that implements `Speak`, automatically implement `Describe` for it".

```
So if

Dog: Speak
Cat: Speak

then it also gets

Dog → Describe
Cat → Describe

without having to individually write

impl Describe for Dog
impl Describe for Cat

"If a type satisfies this condition, automatically give it this behavior."
```

### Associated Types

```
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

`type Item;` is an associated type and the implementation chooses what `Item` is.

```
1.
struct Numbers;

impl Iterator for Numbers {
    type Item = i32;

    // ...
}

2.
struct Names;

impl Iterator for Names {
    type Item = String;

    // ...
}

An associated type says that the implementing type has a particular related type.
```









