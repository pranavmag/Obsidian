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

Brace initialization does have some issues though. In constructor calls, braces and parentheses have the same meaning unless `std::initializer_list` parameters are involved. The compiler prefers the `std::initializer_list` constructors over all others. So if a class has a `std::initializer_list` constructor, the compiler will try to route `{}` calls to it even if perfectly matching standard constructors are there.

It can even hijack normal copy and move constructors if the arguments can be converted to the `initializer_list` type. Compilers only fall back to normal overload resolution if there is absolutely no way to convert the `{}` arguments to the `std::initializer_list` element type.

```
#include <iostream>

  

class ECS {

public:

ECS(int i);

ECS(int i, double d);

ECS(std::initializer_list<bool> il);

};

  

int main() {

ECS e1(10, 2.0); // compiles, it calls the second constructor

ECS e2{10, 2.0}; // compiler error, it calls the initializer_list constructor
// but narrowing conversion is not allowed.

}
```

Because of the hijacking rule, choosing between `()` and `{}` changes the behavior of standard containers like `std::vector`. For example, `std::vector<int> v1(10, 20);` creates a vector with 10 elements all set to the value 20 (using the standard size/value constructor) while `std::vector<int> v2{10, 20};` creates a vector with 2 elements which hold the values 10 and 20 (using the `std::initializer_list` constructor).

The edge case for not being hijacked by the `std::initializer_list` constructor is when there is a default constructor and an empty set of braces `Car c{}` uses the default constructor. If you actually want to pass an empty `initializer_list`, you have to put the braces inside another set of delimiters: `Widget w({});` or `Widget w{{}};`

#### Best Practices

- Class Clients: Pick either `()` or `{}` as your default initialization delimiter and apply it consistently, only breaking the rule when necessary (e.g., using `{}` for container elements or `()` for `std::vector` sizing).
- Class Authors: Be extremely careful when adding `std::initializer_list` constructors to classes. Design your overloads so that users aren't surprised by vastly different behavior depending on whether they use `()` or `{}`.
- **Template Authors:** When forwarding arguments to construct objects (e.g., `std::make_unique` or `std::make_shared`), you cannot know if the caller intended `()` or `{}`. The standard library solves this by defaulting to `()` and explicitly documenting that behavior.


#### Exercise — Constructor selection under review
**Phase/topic:** Phase 2, Item 7  
**Type:** Predict  
**Difficulty:** 2/5  
**Time:** 15–20 minutes

Treat each declaration independently. Assume the necessary headers are included.

```
// A
std::vector<int> a(4, 7);

// B
std::vector<int> b{4, 7};

// C
std::vector<int> c{4};

// D
double reading = 3.75;
int d{reading};

// E
constexpr int count = 12;
unsigned char e{count};
```

Does each compile under C++ 20?

A) Yes it is constructs a vector with 4 elements where each element contains the value 7. 

B) Yes it is constructs a vector with 2 elements with the values 4 and 7 using the `std::initializer_list` constructor.

C) Yes it is constructs a vector with 1 element with the values 4 using the `std::initializer_list` constructor.

D) It does not compile because narrowing conversions are not allowed with brace-initialization.

E) This compiles because the constexpr int value of 12 can be converted to an unsigned char without a narrowing conversion. It wouldn't compile if the value exceeds the maximum of the unsigned char type depending on the system (usually 255) because then it would violate the narrowing conversion rule. It also wouldn't compile if it was a standard int value of 12 because with constexpr the value was known at compile time but not with a regular int so it treats it as though it could be any int value. Since int can lose data in general when converted to an unsigned char it doesn't allow it.

```
struct Settings {
    Settings(int retries, double timeout);
    Settings(std::initializer_list<bool> flags);
};

Settings first(3, 2.5);
Settings second{3, 2.5};
```

Explain which constructor each declaration selects—or why it fails. Specifically address whether the compiler falls back to the `(int, double)` constructor if the initializer-list route requires narrowing.

a) For the `first` object it is constructed with the first constructor successfully because it uses parentheses and the compiler knows that the first constructor matches it.

b) For the `second` object the compiler tries to construct it with the `std::initializer_list` constructor even though the first constructor matches it perfectly. However, it fails to compile because the int value 3 and double value 2.5 cannot be used for a `std::initializer_list<bool>` because it would require narrowing conversion which isn't allowed with brace-initialization. 

