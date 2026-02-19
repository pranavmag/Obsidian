2026-02-10 23:07

Tags: [[functions]] [[void functions]] [[local scope]] [[function parameters]] [[namespaces]] [[unnamed parameters]]
[[preprocessor]] [[header files]] [[header guards]] [[status codes]] [[undefined behavior]]

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








