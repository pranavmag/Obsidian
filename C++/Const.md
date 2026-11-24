2026-11-23 22:47

Tags: [[C++/Learncpp/Chapter 5|Chapter 5]] [[std string SSO and constexpr]] [[Chapter 12]]

const is a compile constraint that an object cannot be modified.

A rule of thumb taken from Bjarne Stroustrup's book "Programming Principles and Practice Using C++" (pg 83 4.3.1 Constant Expressions): If its an expression that is given a value known at compile time, you use 'constexpr'. For variables that are initialized with a value not known at compile time, but never changes after initialization, use 'const'.

```
const int i = 9; // const variable, i cannot be modified
i = 6; // compiler error

const int* p1 = &i // data is const, pointer is not
*p1 = 5; // compiler error, data cannot be modified

std::cout << p1; // output: 0x7ffe01c1c154

p1++; // valid, this increments the pointer by one to the next address

std::cout << p1; // output: 0x7ffe01c1c158

The pointer was incremented by 1 (since it is an int pointer, the address increases by 4 bytes)

int* const p2; // pointer is const, data is not

const int* const p3; // both pointer and data are const
```

It can start to get a bit weird when the placement of keywords and the star can be in random places like `int const *p4 = &i;` vs `const int *p4 = &i;`. In this case, these two are actually the same. The general rules are:

`If const is on the left of *, data is const.`
`If const is on the right of *, pointer is const.`