2026-06-06 20:24

Tags: [[move semantics]] [lvalue] [rvalue] 


### Why Move Semantics Exists

Let's say we have:

Vector a(1000000);
Vector b = a;

Without move semantics, b needs its own memory.

That can be pretty expensive. Instead we could simply steal the memory if we know that a is going to die soon. Just copying an address on over. That's the entire premise of move semantics.


### Lvalues, Rvalues, References

If we have int x = 5;
The variable x is an lvalue.
You can take its address.
You can change its value to x = 10;

rvalues are temporary values like the literal 5.
int x = 40;
int y = x + 10;
y and x are lvalues but the expression x + 10 is an rvalue.

For most cases the definition of lvalues or rvalues in this way is accurate.


Normal references can only bind to lvalues:

int x = 5;
int& ref = x;

int& ref = 5; // illegal because 5 is a temporary


rvalue references:

int&& ref = 5; // this is fine because it can bind to temporaries


std::string&& str = std::string("hello");
This temporary string now has a reference.
If something is temporary we know that no other resource is using it so we can safely steal from it.


### Move Constructor

Deep copy:

Vector(const Vector& other)
{
    size_ = other.size_;

    data_ = new int[size_];

    std::copy(other.data_,
              other.data_ + size_,
              data_);
}


Move constructor:

Vector(Vector&& other)
{
    data_ = other.data_;
    size_ = other.size_;

    other.data_ = nullptr;
    other.size_ = 0;
}

With the second example there is no allocation, no copying, no loops, we just simply move the data on over.

other
     data_ -> heap memory
this
     data_ -> heap memory
other
    data_ -> nullptr

If we didn't set the other data to nullptr then two objects would be pointing to the same data which would cause a double delete when both destructors run which is undefined behavior.


### Move Assignment Operator

Move constructor handles Vector b = std::move(a);

Move assignment handles b = std::move(a);

b already own resources.

Vector& operator=(Vector&& other)
{
    if (this != &other)
    {
        delete[] data_;

        data_ = other.data_;
        size_ = other.size_;

        other.data_ = nullptr;
        other.size_ = 0;
    }

    return *this;
}

We delete the data first because of the fact that b already owns resources so we don't want memory leaks.


### Rule of Five, Three, Zero

The Rule of Three before move semantics was where if you had one of

Destructor
Copy Constructor
Copy Assignment

you'd likely need all three.


Rule of Five is where 

Destructor
Copy Constructor
Copy Assignment
Move Constructor
Move Assignment

if your class manages resources manually then you should be thinking about all five.


The Rule of Zero is where you don't manage resources manually. 

For example if you're using RAII types like vectors or strings the compiler will generate everything automatically so you don't need destructors, copy constructors, or anything.








