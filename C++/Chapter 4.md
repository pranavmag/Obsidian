2026-03-01 18:46

Tags: [[data types]] [[void]] [[signed integers]] [[unsigned integers]] [[floats]] [[booleans]]
[[if statements]] [[chars]] [[type conversion]] [[static_cast]] [[sizeof]]

### Data Types

Data types are the type that an object has before its identifier. Each data type holds its own amount of memory that depends on the architecture of the system. C++ just makes it so that there is a relationship between each one like sizeof(short) <= sizeof(int) <= sizeof(long) <= sizeof(long long), etc. An int on one machine could be 4 bytes whereas on another it could be 2 bytes, so understanding how you can maximize the portability of your code could make it better.



### Void

Void is the one data type that doesn't have a size in bytes, and because of this you can't initialize variables with the data type void. If you do so, the compiler will throw an error because it doesn't know how much memory to allocate for an object that doesn't have a size. Void is most commonly used for functions that don't need to return a value (non-value returning functions). C required you to use void for the parameter of functions that didn't need parameters, but in C++ it is redundant to include void in the parameters.



### Object Size and sizeof Operator

The more memory an object uses, the more information it can hold. You can use the unary operator known as the sizeOf operator to determine the amount of bytes a data type uses. There are integral types that involve data types that use integer values. bool and char are included as integral types because they take in integer values under the hood. Also, just because an object takes less memory doesn't exactly mean that it's faster.



### Signed Integers

Signed integers can represent integers that are negative, positive, and 0. It depends on the architecture for how many signed integers you can use. If you have an 8-bit signed integer the range is from -128 to 127, but if you have a 32-bit signed integer the range is from -2,147,483,648 to 2,147,483,647. as you can see these take in the amount of values for 2^8 and 2^32 respectively. The formula is -(2^n-1) to (2^n-1) - 1. There can also be signed overflow meaning that you go over the range of values of signed integers for the specific architecture. This results in undefined behavior and should be avoided. 



### Unsigned Integers

Unsigned integers can represent integers from the range of 0 and upward. The actual range also depends on the architecture. Unlike signed integers, unsigned integers dont have overflow, they have a wrap around that is not undefined behavior. It is proper C++ behavior used in some situations. It's called modulo arithmetic, for example if you go one over, it wraps to 0 and if you go two over it wraps to 1, because those are the remainders with division.



### Fixed-Width Integers and size_t

Fixed-width integers allow integers to be the same size on every architecture. If you want 32-bit integers, signed or unsigned, you can specify what exactly you want. This is used in some scenarios where it is needed to have a fixed size so the program doesn't crash. when trying to print out 8-bit integers using std::cout it prints out a char value, however when you do arithmetic it works as an 8-bit integer. size_t is a typedef that is simply an unsigned integer. Its size is exactly as big as your computer's architecture. On a 32-bit CPU, size_t is 32 bits, on a 64-bit CPU, size_t is 64 bits.



### Floating Point Numbers

Floating point numbers are numbers that can include a fractional component. These numbers are calculated by the computer through binary so sometimes it may be slightly off. The precision of floating point numbers depends and can vary. Floats are usually 4 bytes and have about 6-8 significant digits of precision whereas a double takes 8 bytes and usually has about 15-17 significant digits of precision. A long double is anywhere from 8-16 bytes and has more significant digits of precision than a double. Never use floating point values for things that need to be exactly precise like in banks. One use of floating point numbers is in video games, where the lost precision cannot be perceived by the human eye so it is negligible. Use floating point numbers if loss of precision is okay and you prefer speed in a program, otherwise use ints and add the fractional component at the end after the computations.


