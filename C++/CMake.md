2026-05-07 18:19

CMake uses configuration file called CMakeLists.txt

a. You define your project in CMakeLists.txt
b. Run CMake to create the Makefile
c. Build your project using Make
d. Add code, fix things, etc then jump back to step c.
e. If you add new .c files or alter the dependencies them jump back to step a.

#### Out-of-source builds

Usually Makefile are placed in the same directory as the source code. You go into the source directory and run Make. 

But CMake is different. The Makefile is generated in a different directory. The CMakeLists.txt file is in the source directory, but the automatically generated build files are separate, so as not to overwhelm your source directory. For example:

/home/nav/src/emulator
   emulator.cpp
   CMakeLists.txt
   build/            <- Run 'cmake ..'
      Makefile
      CMakeCache.txt
      cmake_install.cmake
      CMakeFiles/
      emulator


#### CMakeLists.txt

cmake_minimum_required (VERSION 3.10)

project(emulator)

add_executable(emulator emulator.cpp)

target_link_libraries(emulator PRIVATE m) <- linking math library


#### More

CMake can also generate build files for other build systems, not just Make but also Ninja, Visual Studio, XCode. 

This means the same CMakeLists.txt can be used to generate the files for alternative build systems.



