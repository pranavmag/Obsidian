2026-04-16 23:16

Tags: [[global variables]] [[inline variables]] [[static local variables]] [[inline functions]]

Global variables are variables that are not initialized inside of a function so they have global scope. They are visible from the point of declaration until the end of the file which they are declared in. Global variables have static duration meaning that they are created when the program starts and destroyed when it ends. It is best practice to make global variables constant because if you don't make them constant they can be changed by any function that is called and it's not easy to know that happened. 

Global variables can have external linkage by default and can be made to have internal linkage by using the static keyword, unless the global variables are const or constexpr which have internal linkage by default. To make const or constexpr global variables have external linkage you need to use the extern keyword.

Inline functions are functions that have their code expanded at the point of call at compile time. This is useful for very small and simple functions where making them inline would be useful. In lots of cases they can increase compile times significantly for example if many files are including the header file, the inline function will be compiled in every file before the linker deduplicates the definitions. If that function changes its implementation then every file would need to be recompiled again.

Inline variables are best defined as inline constexpr global variables in a header file. They can be used in constant expressions in any translation unit that includes them and only one copy of each variable is required.

Static local variables make local variables have static duration rather than automatic duration. This means that they are not destroyed when the function ends and can be used again with any amount of function calls made that uses that static local variable. However, they cannot be changed from any other function.