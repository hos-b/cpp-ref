# Modern C++ Reference ([Basic C++](https://github.com/hos-b/cpp-ref))

1. [Types and Stuff](#1-types-and-stuff)<br>
  1.1. [arrays](#11-arrays)<br>
  1.2. [vectors](#12-vectors)
2. [Flow Control](#2-flow-control)<br>
  2.1. [range for loop](#21-range-for-loop)
3. [Unit Tests](#3-unit-test)<br>
  3.1. [CMakeLists.txt](#31-cmakelists.txt)<br>
  3.2. [writing tests](#32-writing-tests)<br>
  3.2. [running tests](#33-running-tests)
4. [Compilation](#4-compilation)<br>
  4.1. [flags](#41-flags)<br>
  4.2. [gdb](#42-gdb)<br>
  4.3. [libraries](#43-libraries)<br>
  4.4. [compilation chain](#44-compilation-chain)
5. [cmake](#5-cmake)<br>
  5.1. [messages, warnings, errors](#51-messages-warnings-errors)<br>
  5.2. [compiler flags](#52-compiler-flags)<br>
  5.3. [using precompiled libraries](#53-using-precompiled-libraries)<br>
  5.4. [building](#54-building)<br>
  5.5. [functions](#55-functions)
  
## 1. Types and Stuff
### 1.1. arrays
`#include<array>`, use `std::array<type, size>`, has constant size
### 1.2. vectors
`#include<array>`, use `std::vector<type>`, unkown amount of items. use `vec.emplace_back()` instead of `vec.push_back()` for efficiency. use `vec.reserve()` for large pushes to avoid consecutive table updates. other useful functions in `#include <algorithm>`:
```cpp
std::sort(vec.begin(), vec.end());    //ascending sort
float sum = std::accumulate (vec.begin(), vec.end(), 0.0f);
float product = std::accumulate (vec.begin(), vec.end(), 1.0f, std::multiplies<float>());
```
## 2. Flow Control
### 2.1 range for loop
iterates over standard containers like `array` or `vector`.
```cpp
for (const auto& value : container){
  //executed for each value in the container
}
```
naturally `const` and `&` are optional.
if container is `std::string` we go over the characters. for two containers, we have to write a normal for loop.
## 3. Unit Tests
install `libgtest-dev`, make a test subdirectory, edit top CMakeLists.txt: 
```cmake
enable_testing()
add_subdirectory(tests)
```
### 3.1. CMakeLists.txt
```cmake
add_subdirectory(/usr/src/gtest
                 ${PROJECT_BINARY_DIR}/gtest)
# include cmake testing package
include(CTest)
set(TEST_BINARY ${PROJECT_NAME}_test)
add_executable(${TEST_BINARY} mylib_test.cpp)
target_link_libraries(${TEST_BINARY}
                      mylib               # libraries we are testing
                      gtest gtest_main    # gtest libraries
)
add_test(
  NAME ${TEST_BINARY}
  COMMAND ${EXECUTABLE_OUTPUT_PATH}/${TEST_BINARY})
```
### 3.2. writing tests
mylib_test.cpp
```cpp
#include <gtest/gtest.h>
#include "mylib.h"

TEST(MylibTest, DummyTest){
  EXPECT_EQ(1, Sum(1,1));
}
```
### 3.3. running tests
```bash
cd build
cmake ..
make
ctest -VV     # very verbose 
```
## 4. Compilation

### 4.1. flags
| flag           | role                   |
|----------------|------------------------|
| -std=c++11     | standard library       |
| -o             | output                 |
| -g             | keep debugging symbols |
| -Wall          | show all warnings      |
| -Wextra        | even more warnings     |
| -Werror        | treat them as errors   |
| -O0            | no optimization        |
| -O3 or -Ofast  | full optimization      |

### 4.2. gdb
build using `-g` flag, `gdb a.out`. use `help`. some commands :

`break main`
`run` or `r`
`print <variable>`
`step` or `s`
#### gdbgui
pip3 install gdbgui; gdbgui a.out
### 4.3. libraries
#### static
`.a`, fast, is incorporated in the final binary. created using `ar rcs <libname.a> <modules>`
#### dynamic
`.so`, slower, referenced by programs.
### 4.4. compilation chain
#### stupid way
compile modules<br>
`c++ -std=c++11 -c <input .cpp files> -o <output .o files>`<br>
organize modules into libraries<br>
`ar rcs libtest.a <.o modules>`<br>
link libraries<br>
`c++ -std=c++11 main.cpp -L <lib path> -ltest -o <executable name>`<br>
run
#### or cmake
defines build receipt
## 5. cmake
```cmake
projec(name)
cmake_minimum_requiired(VERSION 3.1)
set(CMAKE_CXX_STANDARD 11)
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin})
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib})
include_directories(include)
add_library(tools tools.cpp)
add_executable(main main.cpp)
target_link_libraries(main tools)
```
### 5.1. messages, warnings, errors
```cmake
message(STATUS "message")
message(WARNING "message")
message(FATAL_ERROR "message")
```
### 5.2. compiler flags
```cmake
set(CMAKE_CXX_STANDARD 11)
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
set(CMAKE_CXX_FLAGS "-Wall -Wextra")
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")
```
by default, `CMAKE_BUILD_TYPE` is not set.

### 5.3. using precompiled libraries
```cmake
add_library(tools SHARED IMPORTED)
set_property(TARGET tools
             PROPERTY IMPORTED_LOCATION
             "${LIBRARY_OUTPUT_PATH}/libtools.so")
```
### 5.4. building
```bash
mkdir build
cd build
cmake ..
make -j2 #pass number of cores here
```
### 5.5. functions
#### add_subdirectory
add a subdirectory with its own CMakeLists.txt, helps modularity.
```cmake
add_subdirectory(${PROJECT_SOURCE_DIR}/src)
```
src/CMakeLists.txt
```cmake
add_executable(program source.cpp)
```
#### include_directories
additional seach path for header files
```cmake
include_directories(program source.cpp)
```
#### add_library, target_link_libraries
adding source files to be compiled into libraries. afterwards we link them with our executable
```cmake
add_library(libstatic STATIC stat.cpp)
add_library(libdynamic SHARED dyno.cpp)
...
add_executable(program source.cpp)
target_link_libraries(program libstatic
                              libdynamic)
```
