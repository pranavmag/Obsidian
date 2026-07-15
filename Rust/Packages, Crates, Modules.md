2026-07-14 19:30

Tags: 

A package is a bundle of 1 or more crates. it is Cargo's feature to build, test, and share Crates and it contains a Cargo.toml file.

A Crate is a tree of modules and can be a library or an executable. This is the smallest amount of code that the Rust compiler considers at a time.

Modules and use let the programmer control the organization, scope, and privacy of paths.

Paths are a way of naming an item, such as a struct, function, or module.

A crate root is a source file to create the root modules. src/main.rs is the crate root of a binary crate with the same name as the package


### Modules

Modules are defined with the `mod` keyword and they are used to control scope and privacy of the items inside.

