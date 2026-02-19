2026-02-08 02:25

Tags: [[iostream]] [[buffers]] [[initialization]] [[preprocessor]]
[[undefined behavior]] [[literals]] [[operators]] [[expressions]]

preprocessor directives: include iostream essentially allows the program to know what libraries you are using before the source code is written.

### Objects and Variables

Object: a region of storage (usually memory) that allows values to be stored in.
Variables: The name of those objects that are named using an identifier
int width; width is the identifier and the actual object of width can store a value.
initialization of variables
int x;
x = 5;

Copy Initialization int x = 5;

direct initialization int x( 5 );

list initialization: int x{ 5 } ;

value initialization: int x{} -> defaults to 0;

We use list initialization over the other types because it doesn't allow narrowing conversion.
narrowing conversion is for example when the program is expecting an integer but you input a value with a fractional component, it loses the fractional component

There is also something known as the most vexing parse in C++. It is where the compiler can sometimes read int x(5) or something like that and in some situations think that it's a function declaration. You need to let the compiler know that it is an initialization. A very easy solution to that would be to use list-initialization (brace initialization), int x{5}.



### Streams: cout, cin, endl

cout stands for character output, so essentially you are outputting a value to the console. Using the insertion operator you can send the value to the console.

std::cout << "hello world";

cin stands for character input. so essentially you are inputting a value from a user. It uses the extraction operator to allow input to flow.
int x;
std::cin >> x;
there is something also called an input buffer, which essentially acts as a stream, like a water hose, if you let out water and then block it, the remaining water is still there waiting to be released. Likewise in C++, if you get one input from the user, the remaining inputs that you entered will be in the buffer until the next use.
any values in the input buffer are taken as needed as you print it as output. for example if you are taking two integer values and the user enter 3 and 2, those are valid integers so the input buffer checks 3 first deems it as valid and then checks 2 and prints it to the console. If any invalid input is sent then it deems any subsequent value in the input buffer to be invalid and defaults it to 0.


endl essentially moves your cursor to the next line and a better way of doing it is "\n". using \n is better because there is also an output buffer that is for cout, and using endl causes that output buffer to be flushed every time you use endl. flushing the buffer every time is inefficient because you want the buffer to be flushed every so often automatically, not every time. flushing it every time reduces the performance of your program and this is critical in performance heavy programs. \n doesn't flush the output buffer every time.



### Undefined Behavior

Undefined behavior is an outcome that occurs due to executed code not being known by C++ as to how to deal with that. For example, C++ doesn't know what to do with an variable that has no value, so the result with whatever you use that uninitialized variable with may or may not work which is what is known as undefined behavior.



### Literals and Operators

Literals are values that are hard coded into the source code. This is the easiest way to use a value because the CPU doesn't have to retrieve the literal value 5 from the Stack in RAM which would happen with a variable x that is assigned to a value 5 (x = 5). The variable x lives in the Stack which is why the CPU needs to take extra steps to fetch it.

Operators are the symbols that are used for an operation. The inputs that the operator use to produce a new value are called the operands. The amount of operands an operator takes as input is known as the operator's arity. There are a couple types:

Unary operators only act on one operand, for example if we have a value -12, the operator- takes the literal operand 12 and performs a sign change on it to produce the new output value of -12.

Binary operators act on two operands, for example if we have 1 + 2, the operator+ takes the literal operands 1 and 2 to perform mathematical addition on them.

Ternary operators act on three. I do not know any examples of this yet (I don't know if it relates to ternary conditional statements like in Python).

Nullary operators act on zero operands. The throw operator is the only nullary operator and I do not know it's use case for C++ yet (I would think like throw an exception or an error or something but I'm not sure).

Chaining operators is another concept as well. Operators can be changed to perform multiple operations. The output of one operator can be used as the input for another operator. For example, 5 * 2 + 4. The 5 and 2 literal operands are multiplied together by the operator* and that result turns into the operand for the operator+. The two operands for operator+ would be 10 and 4 which are then added together to create a return value of 14.



### Expressions

Expressions are non-empty sequences of literals, function calls, operators, and variables that calculate values.

Another thing is side effects. A lot of times functions, operators, or expressions have side effects other than what they return. A lot of the times, side effects are what we are mainly looking for, for example in void functions, they return nothing, but we simply use them for the side effects that they produce.








