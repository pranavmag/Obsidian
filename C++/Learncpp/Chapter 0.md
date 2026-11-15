2026-02-08 01:35

Tags: [[compiler]] [[linker]] 

This chapter was an introduction to C++.

C++ is a compiled language which means that the code is compiled through a compiler. A C++ compiler is used for this. Some common C++ compilers are compilers like gcc/g++, msvc, and clang.

The fact that C++ is known as a compiled language must mean that there are other categories that other languages fall into. Precisely. Programming languages can either be compiled or interpreted. Some compiled languages are C, C++, and Rust. Some interpreted languages are Python, Ruby, and JavaScript. Some languages like Java choose a hybrid approach of both compiling and interpreting.

What is the main difference though? Compiling drastically increases a program's speed because the code is pre-compiled into machine code as an executable whereas an interpreter directly executes the instructions written in the source code without translating it. Both eventually end up running on hardware. For compiled languages, errors can be caught at compile time which allows for fewer runtime errors. But compiled code is less portable because it is platform specific unless cross-compilation is accounted for. Interpreted code is much slower because each line has to be executed which means that all errors will occur at runtime. But the code is more portable because it can run on any platform with an interpreter.

How does compiling work? So the compiling process is split into a couple steps. First the compiler checks your C++ code to make sure that you are following the rules of C++. Then the compiler translates your C++ code to machine language instructions which are stored in an object file. This is where the linker comes in. A linker combines all your object files the produces an output file like an executable that you can run. Then you can debug your code after that if any errors were generated through that process.

As mentioned earlier, the instructions from the source code are run on hardware. The CPU (central processing unit) is where to instructions are actually executed. Memory is where the programs are loaded before execution. 

