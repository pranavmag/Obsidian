2026-02-10 23:07

Tags: [[functions]] [[void functions]] [[local scope]] [[function parameters]] [[namespaces]] [[unnamed parameters]]
[[forward declarations]] [[preprocessor]] [[header files]] [[header guards]] [[status codes]] [[undefined behavior]]
[[macros]] 

### Functions

Functions are sequences of code that can be reused for just about anything. When declaring a function you have your function header and function body. The function header consists of the function type, the function name, and the parentheses to tell the program that it's a function. The function body consists of the curly braces and any code in between them.

returnType FunctionName() {
	FunctionBody
}

int addTwoNumbers() {
	 code
}

You can make a function call to a certain function in any other functions. So for example if you have your main() function and you want to call the function addTwoNumbers() to use, you can call it from within the function body of main(). The function calling is known as the caller and the function being called is known as the callee. Functions can also call functions that call other functions. However you cannot nest functions within other functions and you cannot call the main() function from other functions.

Functions also have return values. The return type of the function will be the type of the function. We use a return statement to indicate the value being returned to the caller. This is useful because when we call a function from another function, the caller has to know what value to use because we are most likely using that value from the callee.



### Void Functions

Void Functions are non-value returning functions. These functions are used if there isn't a specific value that you need to return. No return statement is required at the end of the function, so there is no need for return; at the end of it because it already does that. However if you include a return value of like 5 or any value then you will get a compile error.

Void functions cannot be used in expressions that require a value. For example if you try to have the main function call the void function within a std::cout << printVoid(); it will not work and you will get a compile error. This is because std::cout expects data to be sitting in a specific CPU register (or stack location) so that it can print it. There will either be no value there or some random value we don't know when it looks, so the compiler will prevent that undefined behavior from happening by generating an error.



### Function Parameters and Arguments

Function parameters are variables that can be initialized on the header of the function. The caller is who specifies that value. For example if you need a function that adds two numbers and you want those two numbers to be initialized in the function header you add those to the parameters. 

int addTwo(int x, int y) {
    return x + y;
}

Something like that works and to call the function you would do addTwo(4, 3). This adds the numbers 4 and 3. The values given by the caller to initialize the variables with are called arguments. The value of each argument is copy initialized into it's corresponding parameter. This is called Pass by Value. For example, addTwo(5, 9), the 5 is sent to the variable of x and 9 is then sent to the variable of y, those numbers get added and the value 14 is returned to the caller.

Unreferenced parameters are parameters that are not used within the body of a function. These will probably generate a warning by the compiler. Unnamed parameters are parameters that don't have a name, so they'll have just the type. These are used when the parameter needs to exist but is not used in the body of the function. These will not generate warnings.



# Local Scope

Scope refers to whether an identifier can be seen and used at points in the code. Local scope means that the identifier is only available to use during the function. Variables and Function parameters are initialized upon the call of the function and those variables and parameters are used throughout the function and they are out of scope once it hits the ending curly brace. Lifetime refers to the amount of time between the creation and destruction of an object. Usually objects are created upon its initialization and destroyed at end of the function.



### Forward Declarations and Definitions

Forward declarations are declarations of a function or variable used to let the compiler know that it exists at some point later in the code or in a different file. Essentially, letting the compiler know about its existence so it doesn't generate an error. A forward declaration for a function involves using a declaration statement:

int add(int x, int y);
or
int add(int, int);

As you can see, the semicolon is necessary because it is a statement. Also, the reason that the parameter names are not needed is because the compiler does not need to know the names. Depending on the compiler, it may mangle the function into a string like Z3addii, the two i's at the end representing two integers.

This is where the linker matters, you can make a forward declaration statement but if you call the function in main() but never actually define the function later with its implementation, the linker will generate an error. Furthermore, if you define the function later but do it wrong, like use the wrong data types or use more parameters than the forward declaration used, the linker can't match the mangled strings properly. 

Take the add function from earlier, if you define it in a different file with

double add(double x, double y) {
     ...
}

the linker will match the two mangled strings, with this one having 'dd' at the end to signify two doubles. The linker is just a dumb matcher, it will generate a linker error because it doesn't know that that is the same function at all because the type of the function changed. However you can change the names of the function or put them in however order:

int add(int y, int x) {
     ...
}

int add(int firstNum, int secondNum) {
     ...
}

Since the compiler never cared about the name, it doesn't tell the linker about it so the linker doesn't care either.



### Namespaces

When you are writing code for a project, you likely have multiple code files, and it would be hard to keep track of function names throughout all the files without knowing. If you have the same function name in multiple files, the code will compile perfectly, but the linker will complain because it will link all definitions in all the files and see a naming collision. This is because these functions are in the same namespace, the global namespace so there is no way to differentiate them. The compiler still compiles it because the compiler doesn't care about the contents of other files in the program while compiling another file, they are all separate translation units. The linker however, links all the object files made by the compiler and turns them into an executable which means that the linker does care about all the files in a program. 

The solution to this is using namespaces. Namespaces provide another type of scope region called a namespace scope that allows you to declare functions and variables within it, but only these, not any executable statements.

void foo(); Allowed
int x; Allowed but discouraged
int x { 5 }; Allowed but discouraged
x= 5; compile error, executable statements are not allowed in namespace

We've actually been using a namespace so far in this journey. std::cout uses the namespace std, which is the standard library's namespace. The actual statement is just cout but to specify what it actually is that we're taking it from, we use the namespace before the scope resolution operator (::). Anything to the left of the scope resolution operator is the namespace and anything to the right of it is the name that is contained within the namespace.



### Preprocessor

The preprocessor is a tool that performs text manipulation on the code file before it is passed to the compiler. It operates on lines that have a hash (#) symbol in it, known as your preprocessor directives. Preprocessor directives start with a hash (#) symbol and end with a newline. They don't end in a semicolon because the preprocessor does not understand C++ syntax. Preprocessors have their own syntax. 

Macros are rules that define how input text is converted into replacement output text. The # define directive can be used to create a macro. There are two basic types of macros: object-like macros and function-like macros. Object-like macros can be defined in one of two ways:

#.define IDENTIFIER
#.define IDENTIFIER substitution text
(No dot is there before define, it's just for obsidian to not tag it)

When there is substitution text, anywhere where the identifier is present will be replaced by the substitution text.

There is also a concept called conditional compilation which allows you to specify under what conditions something compiles or won't compile. There are a few preprocessor directives here: # ifdef, # ifndef, # endif and # if 0. # ifdef checks if an identifier has been previously defined or not, and the # end if is used to end the section where you're checking. # ifndef is the opposite of that obviously. # if 0 is when you just want to comment out a block of code that you can't comment out normally, for example if there's nested comments that aren't possible.



### Header Files

Header files are files that end in the .h extension (sometimes .hpp). These files are used to store forward declarations that are needed and import names from the standard library or user-defined header files. 

You need to be careful in the way in which you include header files that use other header files. Make sure that any dependencies you need aren't depending on the transitive includes.



### Header Guards



