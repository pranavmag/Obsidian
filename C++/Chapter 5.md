2026-03-09 23:13

Tags: [[constants]] [[literals]] [[decimal]] [[binary]] [[hexadecimal]] [[octal]] 
[[compile-time optimization]] [[constexpr]] [[string]] [[string_view]] 

### Constants

constants in the code can take a couple of forms. One of the ones we talked about earlier was just literals in the code. These are hard-coded values in your code and they are not the best to see because they are vague and known as magic numbers in most cases. Try to avoid this unless it's very simple literals that anyone can understand. Another way of doing it is using the const identifier in your code that allows you to set a variable as constant and allow it to not be changed throughout the code. 



### Number Systems

Number systems supported by C++ are decimal, binary, hexadecimal, and octal. Octal isn't used that much but hexadecimal is used for memory addresses. Binary is the language of the computers, known as base 2. Decimal is the number system we are used to, base 10. You can output these by using std::decimal, std::hex, std::octal. Once you use it, your code will use that number system so you have to set it back to decimal after if you were using hexadecimal for example. Binary is a bit different, you need to use std::bitset using the bitset header. 

Digit Separators are using a quotation mark to make it easier to read large numbers. However, you cannot use them before the first digit of the value.

long value {2'132'453'234}; is much easier to read than long value {2132453234};



### Compile-Time Optimization

A program that optimizes another program is called optimizer. The as-if rules says that a compiler can modify a program however it likes to produce more optimized code, as long as those modifications don't affect the program's observable behavior.

Constant Folding is when the compiler replaces literal operands with the result of the expression. Constant Propagation is when the variables that have constant values are replaced with its literal value. Dead code elimination is when the compiler removes code that may be executed but has no effect on the program's behavior.



### Constexpr

Constant expressions only allow integral values, literals, operators, and anything that can be run at compile-time. If you try to make a function call or double value even if it's const, it won't be considered a const expression.


