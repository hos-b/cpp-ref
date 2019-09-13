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
6. [Smart Pointers](#6-smart-pointers)<br>
  6.1.1. [unique pointers](#611-unique-pointers)<br>
  6.1.2. [shared pointers](#612-shared-pointers)<br>
  6.1.3. [examples](#613-examples)
7. [Associative Containers](#7-associative-containers)<br>
  7.1. [map](#71-map)<br>
  7.2. [unordered map](#72-unordered-map)
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
#### finding and including headers
additional seach path for header files
```cmake
find_path(INCLUDE_DIR 
          include/somefile.h 
          <path_to_search_1> <path_to_search_2>)
include_directories(${INCLUDE_DIR})
```
#### add_library
adding source files to be compiled into libraries, to be linked in the same or another project.
```cmake
add_library(libstatic STATIC stat.cpp)
add_library(libdynamic SHARED dyno.cpp)
```
#### finding and linking libraries
```cpp
find_library(LIBS
             NAMES libstatic libdynamic
             PATHS ../lib)

add_executable(program source.cpp)
target_link_libraries(program ${LIBS})
```
#### finding packages
`find_package` calls multiple `find_path` and `find_library` functions. to find a package, cmake must have a file called `Find<pkg>.cmake` in `CMAKE_MODULE_PATH` folders. this variable can be augmented.

`Find<pkg>.cmake` defines which libraries and headers belong to `pkg`. this is predefined for most popular libraries. 

Example `Findsome_pkg.cmake`:
```cmake
find_path(PKG_INCLUDE_DIRS include/smth.h <folder_to_search>)
message (STATUS "headers ${PKG_INCLUDE_DIRS})

find_library(PKG_LIBRARIES
             NAMES some_lib_names
             PATHS <folder_to_search>)
message (STATUS "libraries ${LIBS})
```

## 6. Smart Pointers
they own the object they wrap. `#include <memory>` to use them. they have the same properties as raw pointers :

* can be set to `nullptr`
* `*ptr` for dereferencing
* `ptr->` for member access
* polymorphic

additional functions

* `ptr.get()` returns a raw pointer that the smart pointer manages
* `ptr.reset(raw_ptr)` stops using the current raw pointer, frees its memory if needed, takes owenership of the new `raw_ptr`
### 6.1. unique pointers
constructor of the `unique_pointer` takes ownership of the provided raw pointer. *no runtime overhead over raw pointers*. 
```cpp
#include <memory>
// C++11
auto p = std::unique_ptr<Type>(new Type(<params>));
// C++14 and later
auto p = std::make_unique<Type>(<params>);
```
the copy constructor is explicitly deleted for the unique pointer, however it can be moved e.g. using `std::move`. accessing the unique pointer after move will *most likely* cause a runtime error.

it gurarantees that memory is always owned by a single unique pointer.
### 6.2. shared pointers
just like `unique_pointer` but it can be copied. it also keeps a track of how many shared_pointers have a reference to this pointer. it frees memory when the counter hits zero. It can be initialized from a `unique_pointer`.
```cpp
#include <memory>
// both work since C++11
auto p = std::shared_ptr<Type>(new Type(<params>));
//preferred
auto p = std::make_shared<Type>(<params>);
...
std::cout << "use count : " << ptr.use_count();
```
### 6.3. examples
beginner error : both stack and the smart pointer own the object -> it gets deleted twice and produces errors.
```cpp
int main(){
  int a = 0;
  auto a_ptr = std::unique_ptr<int>(&a);
  return 0
}
```
good use
```cpp
struct Shape{
  ...
}
int main(){
  std::vecotr<std::unique_tr<Shape>> shapes;
  shapes.emplace_back(new Shape);
  shapes.emplace_back(std::make_unique<Shape>());
  auto lvalue_shape = unique_ptr<Shape>(new Shape);
  shapes.emplace_back(std::move(lvalue_shape));
}
```
## 7. Associative Containers

### 7.1. map
stores items under unique keys. usually implemented as red-black tree, i.e. random access in logarithmic time. key can be any type with `operator <` defined. 
```cpp
#include <map>
...
std::map<KeyType, ValueType> m = { {key, value}, {key, value}, {key, value} };
m.emplace(key, value);
// modify or add an item
m[key] = new_value;
// get (const) ref to an item
const auto& = m.at(key);
// check if key is present 
m.count(key)>0
// check size
std::cout << m.size();
// iteration
for (const auto& kv : m) {
  const auto& key = kv.first;
  const auto& value = kv.second;
}
```
one common mistake is using brackets to get const reference to that key. if it's not available, it will be created and initialized based on the default value.
### 7.2. unordered map
same purpose as `std::map` implemented with a hash table. key type has to be hashable, typically `int` or `std::string`. same interface as `std::map`.
