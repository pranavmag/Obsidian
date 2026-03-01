2026-02-22 00:13

Tags: [[syntax errors]] [[semantic errors]] [[debugging]] [[integrated debugger]] 

Syntax errors are errors that occur due to the C++ conventional rules not being followed, such as missing a semicolon and incorrectly typing a keyword. Semantic errors happen when there is a problem with the logic in your code. It is good to practice defensive programming, meaning that you don't trust the output of your code automatically and make sure to code and debug in such a way that you are able to catch errors easily if they occur.

There are a couple ways that you can debug. One of them mentioned in this chapter was to use debug statements. It is recommended to use std::cerr instead of std::cout for these because std::cerr is unbuffered while std::cout is buffered. So std::cout doesn't always output to the screen immediately so you might not know if you're looking in the right or wrong place if you're code crashes before the statement has time to output. A disadvantage to these are that they clutter your code and are extra work because you have to remove these statements. However you can use preprocessor conditional compilation to only allow those debug statements to be used when the conditional compilation allows it. These get rid of the extra work of having to remove those debug statements, but they do add more clutter to your code. 

You can also use a logger. A log is a sequential record of events that have happened, usually time stamped. These logs are usually written to a log file and this is great because it's completely separate from the code. 
P
There is an output stream that C++ provides within the standard library called std::clog that is used to write logging information. However, it writes to the standard error stream (same as std::cerr) and although you can redirect it to file instead, you're better off using a third-party logging tool. One third-party logging tool you can use is the plog logger. 

A debugger allows the programmer to control how a program executes and examines (or modifies if necessary) the program's state. Most IDEs today have an integrated debugger. An integrated debugger uses the same interface as the code editor rather than a separate program that has a command line interface. 

The step into command allows the debugger to go through to every single line of code and stops with each line for the programmer to examine the code's output line by line. When a function is reached the debugger goes to the top of the line of the function that has been called. The step over command does mostly the same as the step into command but instead of executing the function line by line, it executes the whole function and then gives control back to the programmer after it exits the function. The step out command unlike the other two doesn't just execute line by line, rather it executes functions entirely.

A breakpoint is when you can pause execution at a certain point in the code and pass control to the debugger. You can inspect variable values and the program's state at the point of the breakpoint.

The call stack is a feature within integrated debuggers that allow function calls to be tracked through a program. It helps with figuring out function execution order and what is going on with the function calls. Each function call, a new stack frame is created on top of the stack. When a function returns the stack frame is removed and then control goes back to the previous frame. 

