2026-11-14 10:32

Tags: 

When initializing objects there are a couple of ways:

```
int x(0);

int y = 0;

int z{0};

int z = {0}; // C++ usually treats it the same as braces-only version
```

It's important to understand the distinction between initialization and assignment because the use of an equal's sign doesn't always mean assignment. For the fundamental types it is trivial but it matters for user-defined types.

```
Car c1; // calls default constructor

Car c2 = c1; // not an assignment, it calls copy constructor

c1 = c2; // an assignment, it calls copy operator=
```

C++ has uniform initialization, an initialization that in theory can be used anywhere and express everything. It's mostly known as brace initialization. Brace initialization can specify default initialization values for non-statics data members unlike parentheses.

```
class Car {
private:
	int x{0}; // allowed, x's default value is 0
	int y = 0; // also allowed
	int z(0); // NOT allowed
};
```

Also as I learned in learncpp, brace initialization doesn't allow for implicit narrowing conversions among built-in types.

Brace initialization also doesn't have to deal with the most vexing parse. C++'s most vexing parse is when it is ambiguous to the C++ compiler whether or not an initialization of a variable or a function declaration is taking place. `Car c();` may seem like a function named `c` that returns a `Car` object but if braces are used like `Car c{};` there is no ambiguity to the compiler and it can safely call the default constructor to create the object.