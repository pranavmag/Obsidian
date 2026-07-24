2026-07-14 19:30

Tags: 

A package is a bundle of 1 or more crates. it is Cargo's feature to build, test, and share Crates and it contains a Cargo.toml file.

A Crate is a tree of modules and can be a library or an executable. This is the smallest amount of code that the Rust compiler considers at a time.

Modules and use let the programmer control the organization, scope, and privacy of paths.

Paths are a way of naming an item, such as a struct, function, or module.

A crate root is a source file to create the root modules. src/main.rs is the crate root of a binary crate with the same name as the package


### Modules

Modules are defined with the `mod` keyword and they are used to control scope and privacy of the items inside.

The compiler always starts by looking at the crate root file (src/lib.rs for a library crate or src/main.rs for a binary crate) for code to compile.

In the crate root file (main.rs), you can declare new modules. For example you declare a "garden" module with `mod garden;`

The compiler will look for the module's code:
	Inline (curly brackets replace the semicolon following `mod garden`)
	In the file src/garden.rs
	In the file src/garden/mod.rs

Submodules can be declared in files other than the crate root. For example, declare `mod vegetables;` in src/garden.rs.

The compiler will look for the submodule's code within the dir named for the parent module:
	Inline, following `mod vegetables` (within curly brackets instead of the semicolon)
	In the file src/garden/vegetables.rs
	In the file src/garden/vegetables/mod.rs

Once a module is part of your crate, you can refer to code in that module from anywhere else in that same crate using the path to the code. 

The use keyword creates shortcuts to items to reduce repetition of long paths. Instead of typing every time `crate::garden::vegetables::Celery` for each instance we use `Celery`, we can say `use crate::garden::vegetables::Celery;` at the top of the file and then use just `Celery` to refer to it. 

By default, the code within a module is private from its parent modules. To make a module public, declare it with `pub mod` instead of `mod`. To make items within a public module public as well, use pub before their declarations.

You can use modules to organize code for better readability, but also to separate responsibilities.


### Paths

```
crate::garden::vegetables::get_celery(); // Absolute Path

garden::vegetables::get_celery(); // Relative Path
```

We can use `super` to access a parent module.

```
fn deliver_order() {
	// function body
}

mod back_of_house {
	fn fix_incorrect_order() {
		cook_order();
		super::deliver_order();
	}
	
	fn cook_order() {}
}
```

As you can see we can access a function outside of the module by using the `super` keyword.