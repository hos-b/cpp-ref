# Modern C++

1. [Types and Stuff](#1-types-and-stuff)<br>
  1.1. [arrays](#11-arrays)<br>
  1.2. [vectors](#12-vectors)
2. [Flow Control](#2-flow-control)<br>
  2.1. [range for loop](#21-range-for-loop)
10. [Compilation](#10-compilation)<br>
  10.1. [flags](#101-flags)
  
## 1. Types and Stuff

### 1.1. arrays
`#include<array>`, use `std::array<type, size>`, has constant size
### 1.2. vectors
`#include<array>`, use `std::vector<type>`, unkown amount of items. use `vec.emplace_back()` instead of `vec.push_back()` for efficiency. use `vec.reserve()` for large pushes to avoid consecutive table updates. other useful functions:

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

## 10. Compilation

### 10.1. flags
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


### 10.2. gdb
build using `-g` flag, `gdb a.out`. use `help`. some commands :

`break main`
`run` or `r`
`print <variable>`
`step` or `s`

#### gdbgui
pip3 install gdbgui; gdbgui a.out

### 10.3. libraries
#### static
`.a`, fast, is incorporated in the final binary. created using `ar rcs <libname.a> <modules>`
#### dynamic
`.so`, slower, referenced by programs.
### 10.4. compilation chain
#### stupid way
comile modules
`c++ -std=c++11 -c <input .cpp files> -o <output .o files>`
organize modules into libraries
`ar rcs libtest.a <.o modules>`
link libraries
`c++ -std=c++11 main.cpp -L <lib path> -ltest -o <executable name>`
run
#### cmake
defines build receipt
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
messages, warnings, errors
```cmake
message(STATUS "message")
message(WARNING "message")
message(FATAL_ERROR "message")
```
flags
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

using precompiled libraries
```cmake
add_library(tools SHARED IMPORTED)
set_property(TARGET tools
             PROPERTY IMPORTED_LOCATION
             "${LIBRARY_OUTPUT_PATH}/libtools.so")
```
building:
```bash
mkdir build
cd build
cmake ..
make -j2 #pass number of cores here
```
