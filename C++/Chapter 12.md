2026-05-26 21:12

Tags: [[lvalue]] [[rvalue]] [[references]] [[pointers]]

### lvalues and rvalues

lvalues are any values that refer to an identifiable object and rvalues are any values that can't be reused after the expression they are currently being used in such as literals (except C-style string literals). This is because rvalues are temporaries. For example:

int x = 5;
x is an lvalue but the literal 5 is an rvalue.
You can take the address of (&) x but you can't take the address of (&) the literal 5 because the address of operator to lvalues only (most of the time).

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

int add (int& x, int& y) {implementation}

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


### Pointers

Pointers are variables that store the memory address of other objects or functions. You can instantiate a pointer with this syntax:

int x = 2;
int* ptr = &x;

Pointers unlike references can be reseated to point to a different memory address and like references they can change the value of the object held at that memory address. Accessing or changing the value of the object is known as dereferencing the pointer. I will show an example of each respectively:

ptr = &y;  // reseats pointer to point to the memory address of y
std::cout << *ptr;  // dereference the pointer and print the value
*ptr = 4;    // dereferences pointer to change value of the object stored at the memory address the pointer it pointing to

Pointers provide more flexibility than references but they can be more dangerous as well. Unlike references which can't point to null, pointers can point to null, which we call null pointers. References must point to valid objects. A null pointer just means that the pointer is pointing to no object. You can't dereference a null pointer because there is no object to dereference the value of.

Pointers can also be dangling pointers. This happens when the object you are pointing to is destroyed before the pointer can be destroyed. This is dangerous because there is no easy way to know this happened because it is undefined behavior. In the case of a nullptr that wasn't intended, you can check it using an if statement. But, in the case of a dangling pointer, it may be pointing to an object, just not what you intended so it's hard to tell. It is much harder to get a dangling reference so references are generally seen as safer for most situations.

Pointers can also be pointing to a const value and also be const pointers. These two are different things:

const int* ptr = &x;    // pointer to a const object
int* const ptr = &x;   // const pointer to an object

A pointer to a const object means that you cant change the value of the object but you can reseat the pointer, the pointer is not const but the object it is pointing to is. 

A const pointer can change the object's value but the pointer cannot be reseated to point to something different.

If you want both then do:

const int* const ptr = &x;   // This makes both the pointer and the underlying object const.


### Pass by Address

Pass by Address allows you to take in a pointer parameter in functions. What it essentially does is copy the address of the object you pass in and create a ptr. This means that if you reseat the pointer to a different value within the function, it won't reseat the pointer from the caller. Although changing the value of the object at that memory address will change the object that the caller's ptr is looking at as well as long as they are both pointing to that memory address. It's important to make that distinction. There are two ways to pass by address:

void increment(int* ptr) {implementation}

int main() {

int x = 5;
int* ptr = &x;

increment(ptr);  // Passing in the pointer directly, ptr in increment's function parameter makes a new pointer copy to the same address

increment(&x);  // Passing in the reference to make the pointer
}

