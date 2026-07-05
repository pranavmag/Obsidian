2026-07-04 12:26

Tags: 

Structs are very similar to tuples but they have named fields. They can be used to create more complex data types.

```
#[derive(Debug)]  
struct User {  
    name: String,  
    email: String,  
    is_active: bool,  
    age: u8  
}  
  
fn main() {  
    let user1 = User {  
        name: String::from("Nav"),  
        email: String::from("nav@dbint.db"),  
        is_active: true,  
        age: 25  
    };  
  
    println!("{:?}", user1);  
}
```

```
User { name: "Nav", email: "nav@dbint.db", is_active: true, age: 25 }
```

Structs use key-value stores it seems like. Also I noticed that we can print the whole struct just like that by using the debug trait.

```
#[derive(Debug)]  
struct User {  
    name: String,  
    email: String,  
    is_active: bool,  
    age: u8  
}  
  
fn main() {  
    let user1 = User {  
        name: String::from("Nav"),  
        email: String::from("nav@dbint.db"),  
        is_active: true,  
        age: 25  
    };  
  
    user1.name = String::from("aajonus"); // compiler error
  
    println!("{:?}", user1);  
}
```

Structs are immutable by default unless you make the variable mutable. You can't make a single field mutable unfortunately. Either the instance as a whole is mutable or immutable.

```
#[derive(Debug)]  
struct User {  
    name: String,  
    email: String,  
    is_active: bool,  
    age: u8  
}  
  
fn build_user(name: String, email: String) -> User {  
    User {  
        name,  
        email,  
        is_active: false,  
        age: 66  
    }  
}  
  
fn main() {  
    let user1 = build_user(String::from("Aajonus Vonderplanitz"), String::from("aaj@gmail.com"));  
}
```

You can also create structs from functions.


### Tuple Structs

```
struct Color(i32, i32, i32);  
struct Point(i32, i32, i32);  
  
fn main() {  
    let red = Color(255, 0, 0);  
    let origin = Point(0, 3, 4);  
}
```

I think this could be helpful if we want to know what our tuple fields could represent. When we see color and see a tuple struct of three values we know we are using RGB values, so it makes it very useful to have it like that.