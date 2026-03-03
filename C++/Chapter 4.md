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

Signed integers can represent integers that are negative, positive, and 0. It depends on the architecture for how many signed integers you can use. If you have an 8-bit signed integer the range is from -128 to 127, but if you have a 32-bit signed integer the range is from -2,147,483,648 to 2,147,483,647. as you can see these take in the amount of values for 2^8 and 2^32 respectively. The formula is -(2^n-1) to (2^n-1) - 1.

