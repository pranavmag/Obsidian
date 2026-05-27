2026-05-26 21:12

Tags: 

### lvalues and rvalues

lvalues are any values that can be reused. rvalues are any values that can't be reused after the expression they are currently being used in such as literals (except C-style string literals). For example:

int x = 5;
x is an lvalue but the literal 5 is an rvalue.
You can take the address of (&) x but you can't take the address of (&) the literal 5 because the literal 5 is not a valid object that has a memory address. 

Why is a C-style string literal not a r-value reference? This is because a C-style string literal has the possibility of decaying to a const char array pointer.

### lvalue references and const lvalue references

lvalue references are any references that you pass to an object. If you have a variable x and you define a new variable that is &y = x, then you have a reference to x. You can also use references in function parameters.

For example if you have a function:

int add(x, y) {implementation}

and you call it from main()

int main {
int a{23};
int b{20};
add(a, b);
}

if you increment the values of x and y there, they do not affect the values of a and b in main. This is because x and y are different objects than a and b. Unless you pass in by reference like:

int add (&x, &y) {implementation}

int main() {
int a{23};
int b{20};
add(a, b);
}

In this case, the values of a and b will be incremented if you increment x and y because now x and y are pointing to the objects a and b so any operation performed on the parameters will reflect back to the objects of the caller.

Function parameter references are just pointers under the hood.

Although these need to be objects, you cannot pass in literal values to the function call when taking a reference.

int add (&x, &y) {implementation}

int main() {
add(23, 20);    // not allowed
}

This is because lvalue references need an object to point to. Unless you use a const lvalue reference.

Const lvalue references are allowed to point to rvalues. So you can have

int add (const &x, const &y) {implementation}

int main() {
add(23, 20);    // allowed
}

Why is this the case?

This is because you are just observing a value with the const lvalue reference and telling the compiler that it shouln't be modified, which is okay to do on a temporary. Another thing is because the temporary object will not be destroyed until the reference is either as to not cause a dangling reference. The temporary's lifetime is extended to match the const lvalue reference. This doesn't happen with non-const lvalue references because you'll be passing in a valid object in memory.


### Pass by Value vs Pass by Reference

When should you pass by value and when should you pass by reference?

The general rule of thumb is to pass by value if the object is cheap to copy, where the benefits of passing by reference are outweighed. There is a cost of passing by reference but if the object is expensive enough, such as vectors, strings, structs, then passing by reference is generally preferred. For an integer that is always 4 bytes, or any other primitive types it is preferred to pass by value.

But in the case of strings, you can pass in a const std::string& but the better approach is to pass by value here using a std::string_view. Passing by value with a std::string view offers 2 more cases where you can use it. Lets say we have these two parameters and pass in three different values: std::string, std::string_view, and C-style string literal. 

For the const std::string&, the std::string is an inexpensive reference binding, however the std::string_view is an expensive explicit conversion to std::string, and the C-style string literal is an expensive conversion. For the std::string_view parameter, the std::string is a inexpensive conversion, the std::string_view is an inexpensive copy, and the C-style string literal is an inexpensive conversion. You get two more cases covered inexpensively by using a std::string_view parameter.

