Modern C++ Reference ([Basic C++](https://github.com/hos-b/cpp-ref))

1. [Types and Stuff](#1-types-and-stuff)<br>
  1.1. [arrays](#11-arrays)<br>
  1.2. [vectors](#12-vectors)
2. [Flow Control](#2-flow-control)<br>
  2.1. [range for loop](#21-range-for-loop)
3. [Unit Tests](#3-unit-tests)<br>
  3.1. [CMakeLists.txt](#31-cmakelists.txt)<br>
  3.2. [writing tests](#32-writing-tests)<br>
  3.3. [running tests](#33-running-tests)
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
6. [Move Semantics](#6-move-semantics)<br>
  6.1. [lvalue and rvalue](#61-lvalue-and-rvalue)<br>
  6.2. [example](#62-example)<br>
  6.3. [move constructor and assignment](#63-move-constructor-and-assignment)
7. [Smart Pointers](#7-smart-pointers)<br>
  7.1. [unique pointers](#71-unique-pointers)<br>
  7.2. [shared pointers](#72-shared-pointers)<br>
  7.3. [weak pointers](#73-weak-pointers)<br>
  7.4. [examples](#74-examples)
8. [Associative Containers](#8-associative-containers)<br>
  8.1. [map](#81-map)<br>
  8.2. [unordered map](#82-unordered-map)
9. [Iterators](#9-iterators)<br>
  9.1. [properties](#91-properties)<br>
  9.2. [examples](#92-examples)
10. [CPP Lambdas](#10-cpp-lambdas)<br>
  10.1. [definition](#101-definition)<br>
  10.2. [function pointers](#102-function-pointers)<br>
  10.3. [std examples](#103-std-examples)<br>
  10.4. [recursive lambdas](#104-recursive-lambdas)<br>
  10.5. [stateful lambdas](#105-stateful-lambdas)
11. [Multi-Threading](#11-multi-threading)<br>
  11.1. [daemon processes](#111-daemon-processes)<br>
  11.2. [passing parameters](#112-passing-parameters)
12. [Mutex](#12-mutex)<br>
  12.1. [lock_guard](#121-lock_guard)<br>
  12.2. [multiple mutexes](#122-multiple-mutexes)<br>
  12.3. [unique lock](#123-unique-lock)<br>
  12.4. [sleeping and condition variable](#124-sleeping-and-condition-variable)<br>
  12.5. [future](#125-future)<br>
  12.6. [promise](#126-promise)<br>
  12.7. [callable objects](#127-callable-objects)<br>
  12.8. [packaged tasks](#128-packaged-tasks)<br>
  12.9. [time constraints](#129-time-constraints)
13. [OpenCV](#13-opencv)<br>
  13.1. [basic matrix type](#131-basic-matrix-type)<br>
  13.2. [memory management](#132-memory-management)<br>
  13.3. [IO](#133-io)<br>
  13.4. [vector type](#134-vector-type)<br>
  13.5. [SIFT descriptors](#135-sift-descriptors)<br>
  13.6. [FLANN](#136-flann)
14. [bind](#14-bind)<br>
  14.1. [binding with reference](#141-binding-with-reference)<br>
  14.2. [binding arbitrary arguments](#142-binding-arbitrary-arguments)<br>
  14.3. [binding with arbitrary order](#143-binding-with-arbitrary-order)<br>
  14.4. [binding with template functions](#144-binding-with-template-functions)
15. [C++14](#15-c++14)<br>
  15.1. [exchange](#151-exchange)
16. [C++17](#16-c++17)<br>
  16.1. [structured bindings](#161-structured-bindings)<br>
  16.2. [any](#162-any)<br>
  16.3. [if and switch initialization](#163-if-and-switch-initialization)<br>
  16.4. [nested namespaces](#164-nested-namespaces)<br>
  16.5. [has include](#165-has-include)<br>
  16.6. [aggregate initialization](#166-aggregate-initialization)
17. [C++20](#17-c++20)<br>
18. [Attributes](#18-attributes)<br>
  18.1. [fallthrough](#181-fallthrough)<br>
  18.2. [unused](#182-unused)<br>
  18.3. [maybe_unused](#183-maybe_unused)<br>
  18.4. [nodiscard](#184-nodiscard)
99. [Misc.](#99-misc.)<br>
  99.1. [timing](#991-timing)<br>
  99.2. [variadic templates](#992-variadic-templates)<br>
	99.3. [next](#993-next)<br>



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
## 6. Move Semantics

### 6.1. lvalue and rvalue
* an expression is an lvalue when *it can* be written on the left side of the assignment operator `=`.
* other expressions are rvalues. 
* explicit rvalue defined using `&&`
* `std::move()` used to convert an lvalue to an rvalue
```cpp
int a;                    // a = lvlaue
int& a_ref = a;           // a_ref = reference to lvalue, a = lvalue
a = 2+2;                  // a = lvlaue, 2+2 = rvalue
int b = a+2;              // b = lvalue, a+2 = rvalue
int &&c = std::move(a);   // c = rvalue
```
typically when using rvalues in functions, we want to move ownership, therefor using `const <type>&&` makes no senese.
### 6.2. example
```cpp
void Print (const std::string& str){
  std::cout << "lvalue : " << str;
}
void Print (std::string&& str){
  std::cout << "rvalue : " << str;
}
int main(){
  string hello = "hi";
  Print(hello);               // lvalue
  Print("world");             // rvalue
  Print(std::move(hello));    // rvalue, should not access hello anymore
  return 0;
}
```
by c++ standard, after an object has been moved, it should be empty. moving a variable transfers ownership of its resource to another variable. runtime is better than copying, worse than passing by reference.
### 6.3. move constructor and assignment
```cpp
MyClass::MyClass (MyClass&&);   // move-constructor
MyClass& operator= (MyClass&&); // move-assignment
```
move constructor is automatically called when the object is moved.<br>
move assignment is automatically called when the object is assigned a new value from an rvalue.
```cpp
MyClass a;
MyClass b(std::move(a));    // move constructor
MyClass c = std::move(a);   // move constructor
b = std::move(c);           // move assignment
```
the last syntax explicitly calls the move assignment, however compilers tend to recognize when a copy assignment is needed, specially when a big object returned from a function is being assigned to a variable.
## 7. Smart Pointers
they own the object they wrap. `#include <memory>` to use them. they have the same properties as raw pointers :

* can be set to `nullptr`
* `*ptr` for dereferencing
* `ptr->` for member access
* polymorphic

additional functions

* `ptr.get()` returns a raw pointer that the smart pointer manages
* `ptr.reset(raw_ptr)` stops using the current raw pointer, frees its memory if needed, takes owenership of the new `raw_ptr`
### 7.1. unique pointers
constructor of the `unique_pointer` takes ownership of the provided raw pointer. *no runtime overhead over raw pointers*. 
```cpp
#include <memory>
// C++11
auto p = std::unique_ptr<Type>(new Type(<params>));
// C++14 and later
auto p = std::make_unique<Type>(<params>);
```
the copy constructor is explicitly deleted for the unique pointer, however it can be moved e.g. using `std::move`. accessing the unique pointer after move will *most likely* cause a runtime error.<br>
it gurarantees that memory is always owned by a single unique pointer. to point to a new object use `ptr.reset(<new pointer>)`, most likely with `std::move` to take ownership. <br>
the `std::make_unique` constructor is preferred for exception safety.
### 7.2. shared pointers
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
it's much more efficient to use `std::make_shared` because shared pointers also have a control block that keeps track of the reference count. if the `new` keyword is used, the control block and they type will be created seperately, but with `std::make_shared` they're created together.
### 7.3. weak pointers
this behaves the same as an extra instance of shared pointer without increasing the reference count. we would use this kind of pointer when we want access to the raw pointer, but don't want to take ownership.
```cpp
class Test{
public:
  Test() { std::cout << "created\n";}
  ~Test() { std::cout << "destroyed\n";}
}
int main(){
  {
    std::weak_pointer weak;
    {
      std::shared_pointer<Test> shared = std::make_shared<Test>();
      weak = shared;
      std::cout << "original object managed by : "  << weak.use_count() << std::endl;
    }
    std::cout << "original object alive : "  << weak.expired() << std::endl;
  }
}
```
#### changed my mind
we can use `weak_ptr.lock()` which returns a shared_ptr which shares ownership of the owned object if std::weak_ptr::expired returns false. Else returns default-constructed shared_ptr of type T. 
### 7.4. examples
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
## 8. Associative Containers

### 8.1. map
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
### 8.2. unordered map
same purpose as `std::map` implemented with a hash table. key type has to be hashable, typically `int` or `std::string`. same interface as `std::map`.
#### example
here we can see what the default hash function does to a string :
```cpp
int main(int argc, char* argv[])
{
    cout << std::hash<const char *>()("Mark Nelson") << endl;
    return 0;
}
output : 134514544
```
here's an example with custom key and hash fucnction
```cpp
size_t name_hash( const std::pair<string, string> &name)
{
    return hash<string>()(name.first) ^ hash<string>()(name.second);
}

int main(int argc, char* argv[])
{
	unordered_map<std::pair<string, string>,int ,decltype(&name_hash)> ids(100, name_hash );
	ids[std::pair<string, string>("Mark", "Nelson")] = 40561;
	ids[std::pair<string, string>("Andrew","Binstock")] = 40562;
	for ( auto ii = ids.begin() ; ii != ids.end() ; ii++ )
		cout << ii->first.first 
                     << " "
                     << ii->first.second 
                     << " : "
                     << ii->second
                     << endl;
	return 0;
}
```
more info [here](https://marknelson.us/posts/2011/09/03/hash-functions-for-c-unordered-containers.html).
## 9. Iterators
a type that iterates over a container. it can be regarded as a pointer:
* access current element through '*iter*
* accepts `->` operatpr
* moves to the next element using `iter++`
### 9.1 properties 
* comparable with ==, !=, <
* predefined iterators in STL : obj.begin(), obj.end()
* range-based for loops preferred
`begin()` points to the start of the container. `end()` points to one element after the last element, i.e. nothing
### 9.2 examples
#### simple iterator loops
```cpp
std::vector<double> x = {{1,2,3}};
for (auto it = x.begin(), it != x.end(); ++it){
    std::cout << *it << std::endl;
}
```
#### map iterator
```cpp
std::map<int, std::string> m = {{1, "hello"}, {2, "world"}};
std::map<int, std::string>::iterator m_iter = m.find(1);
std::cout << m_iter->first << ":" << m_iter.second << endl;
if (m.find(3) == m.end())
  std::cout<< "key 3 not found\n" ;
```
in the case of `std::map` it's much more readable to write `if(m.count(3))` instead.
## 10. CPP lambdas

a simple example using `std::foreach` :
```cpp
vector<int> v;
v.push_back(...);
...
int evenCount=0;
for_each(v.begin(), v.end(), [&evenCount] (int n) {
      cout << n;
      if (n % 2 == 0) {
         cout << " is even " << endl;
         ++evenCount;
      } else {
         cout << " is odd " << endl;
      }
   });
```
### 10.1. definition
lambdas are made of 6 parts : 
```cpp
[] () mutable throw() -> int
{
  
}
```
1. capture clause (Also known as the lambda-introducer in the C++ specification.)
2. parameter list Optional. (Also known as the lambda declarator)
3. mutable specification Optional.
4. exception-specification Optional.
5. trailing-return-type Optional.
6. lambda body.

#### capture clause
A lambda can introduce new variables in its body (in C++14), and it can also access, or capture, variables from the surrounding scope. A lambda begins with the capture clause (lambda-introducer in the Standard syntax), which specifies which variables are captured, and whether the capture is by value or by reference. Variables that have the ampersand `&` prefix are accessed by reference and variables that do not have it are accessed by value. An empty capture clause, `[ ]`, indicates that the body of the lambda expression accesses no variables in the enclosing scope.
`[&]` means all variables that you refer to are captured by reference, and `[=]` means they are captured by value. You can use a default capture mode, and then specify the opposite mode explicitly for specific variables. For example, if a lambda body accesses the external variable total by reference and the external variable factor by value, then the following capture clauses are equivalent:
```cpp
[&total, factor]
[factor, &total]
[&, factor]
[factor, &]
[=, &total]
[&total, =]
```
in C++14, you can introduce and initialize new variables in the capture clause, without the need to have those variables exist in the lambda function’s enclosing scope. 
#### parameter list
in addition to capturing variables, a lambda can accept input parameters. A parameter list is optional and in most aspects resembles the parameter list for a function. in C++ 14, if the parameter type is generic, you can use the auto keyword as the type specifier.
```cpp
auto y = [] (int first, int second)
{
    return first + second;
};
```
#### mutable
typically, a lambda's function call operator is const-by-value, but use of the mutable keyword cancels this out. It does not produce mutable data members. The mutable specification enables the body of a lambda expression to modify variables that are captured by value. if we don't use `mutable` we'll get a compilation error.
#### exception specification
you can use the noexcept exception specification to indicate that the lambda expression does not throw any exceptions. As with ordinary functions, the compiler generates warning C4297 if a lambda expression declares the noexcept exception specification and the lambda body throws an exception.
#### return type
the return type of a lambda expression is automatically deduced. You don't have to use the auto keyword unless you specify a trailing-return-type. The trailing-return-type resembles the return-type part of an ordinary method or function. However, the return type must follow the parameter list, and you must include the trailing-return-type keyword `->` before the return type.

You can omit the return-type part of a lambda expression if the lambda body contains just one return statement or the expression does not return a value. If the lambda body contains one return statement, the compiler deduces the return type from the type of the return expression. Otherwise, the compiler deduces the return type to be void.
```cpp
auto x1 = [](int i){ return i; }; // OK: return type is int
auto x2 = []{ return{ 1, 2 }; };  // ERROR: return type is void, deducing
                                  // return type from braced-init-list is not valid
```
#### body
The lambda body (compound-statement in the Standard syntax) of a lambda expression can contain anything that the body of an ordinary method or function can contain. The body of both an ordinary function and a lambda expression can access these kinds of variables:
1. Captured variables from the enclosing scope, as described previously.
2. Parameters
3. Locally-declared variables
4. Class data members, when declared inside a class and this is captured
5. Any variable that has static storage duration—for example, global variables
```cpp
int main()
{
   int m = 0;
   int n = 0;
   [&, n] (int a) mutable { m = ++n + a; }(4); //m=5, n=0
}
```
The `mutable` specification allows n to be modified within the lambda.
### 10.2. function pointers
we can use lambdas wherever a function pointer is needed. e.g. here ForEach function needs a function pointer
```cpp
#include <algorithm>
#include <iostream>
#include <vector>

void ForEach(const std::vector<int>& values, void (*func)(int))
{
  for (int value : values)
    func(value);
}
int main()
{
  std::vector<int> values = {1,5,4,3,2};
  ForEach(values, [](int value) { std::cout << "Value : " << value << std::endl;});
}
```
we can also use `auto` to make pasing lambdas cleaner:
```cpp
auto lambda = [](int value) { std::cout << "Value : " << value << std::endl;};
ForEach(values, lambda);
```
this is only doable when we don't want to pass any of the local variables to the lambda. if we do, then we can no longer use a traditional function pointer. instead we use `std::function<return_type(param types)& func`
```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <functional>
void ForEach(const std::vector<int>& values, const std::function<void(int)>& func)
{
  for (int value : values)
    func(value);
}
int main()
{
  std::vector<int> values = {1,5,4,3,2};
  ForEach(values, [=](int value) { std::cout << "Value : " << value << std::endl;});
}
```
### 10.3. std examples
a lot of std functions take lambdas. 
#### find_if
```cpp
std::vector<int> values = {1,5,4,3,2};
auto iter = std::find_if(values.begin(), values.end(), [](int value) { return value > 3; });
std::cout << *iter << std::endl;
```
this finds the first element that is larger than 3, i.e. 5.
#### sort
```cpp
std::vector<int> vec = {10, 21, 3, 19, 32};
std::sort(vec.begin(), vec.end(), [] (int x, int y) { return x<y ;});
```
#### copy_if
```cpp
std::vector<int> vec = {10, 21, 3, 19, 32};
std::vector<int> even_vec;
std::copy_if(vec.begin(), vec.end(), std::back_inserter(even_vec),
              [] (int x) { return (x%2)==0 ;});
```
#### foreach
```cpp
std::vector<int> vec = {10, 21, 3, 19, 32};
int sum = 0;
std::for_each(vec.begin(), vec.end(), [&sum](int x){sum+=x;});
```
#### transform
summing up two vectors:
```cpp
std::vector<int> vec1 = {10, 21, 3, 19, 32};
std::vector<int> vec2 = {10, 21, 3, 19, 32};
std::vector<int> vec3;
std::transform(vec1.begin(), vec1.end(), vec2.begin(), vec3.begin(),
              [](int x,int y){return x+y;} );
```
### 10.4. recursive lambdas
by passing the reference of the `std::function` back to its lambda
```cpp
std::function<int(int)> Fib = [&Fib](int n) 
                      { return n < 2 ? n : Fib(n - 1) + Fib(n - 2);};
std::cout << "Fib(4) :" << Fib(4);
```
### 10.5. stateful lambdas
when values are captured by copy, they are not mutable unless the lambda is defined as one. afterwards they can be used as a state:
```cpp
int main()
{
  int i = 1;
  auto lambd = [i]() mutable {return ++i;};
  lambd();
  return lambd(); // returns 3
}
```
we can also skip the variable definition and directly put it in the capture clause:
```cpp
int main()
{
  auto lambd = [i = 1, ptr = std::unique_ptr<int>(3)]() mutable {return ++i;};
  lambd();
  auto lamb2 = lambd;   // not allowed, unique_ptrs cannot be copied
  return lambd();       // returns 3
}
```
## 11. Multi-Threading
simple example
```cpp
#include <iostream>
#include <thread>

void thread_func(){
  std::cout << "thread output" << std::endl;
}
int main(){
  std::thread t1(thread_func);  // create and start thread
  std::cout << "thread id : " << t1.get_id() << std::endl;
  t1.join(); // wait for thread to finish
  return 0;
}
```
#### note
threads cannot be copied because the copy constructor is explicitly removed. we can however transfer ownership using `std::move`.
### 11.1. daemon processes
usually it's on the main thread to reclaim the resources allocated to other threads. if a thread is supposed to run for a longer time, we detatch the thread. the c++ runtime library will be responsible for freeing the resources.
```
int main(){
  std::thread t1(thread_func);  // create and start thread
  t1.detatch();
  return 0;
}
```
a detatched thread is also referred to as a _daemon process_. they may run until system restart. <br>
after a process has been detatched, it cannot be joined again. to check if it's possible to join a thread, we can use `thread.joinable()`

### 11.2. passing parameters
```cpp
#include <iostream>
#include <thread>

class Functr{
  void operator()(string &msg){
    std::cout << "functor arg : " << msg << std::endl;
    msg = "printer by functor";
  }
};

void thread_func(string &msg){
  std::cout << "function arg" << std::endl;
  msg = "printed by function";
}
int main(){
  string msg = "hello";
  std::thread t1(thread_func, msg);
  t1.join();
  std::cout << msg << std::endl;
  std::thread t2((Functr()), std::ref(msg));
  t2.join();
  std::cout << msg << std::endl;
  return 0;
}
```
normally all function parameters are passed by value, even if explicitly defined as reference. to pass the arguement as reference, additionally we have to use `std::ref(arg)`. <br>
another aproach would be to just share the pointer to the object. if there is no need to share the object between threads, it's good practice to transfer its ownership to the thread using `std::move(arg)`.
#### note
the functor is additionally wrapepd in parentheses because otherwise it could be interpreted as a function declaration. in general anything that can be regarded as function declaration, will.
### 11.3. oversubscription
generally we make as many threads as we have cpu cores. to get a hint about how many is recommended, we can use the value of `std::thread::hardware_concurrency`.

## 12. Mutex
to control access to shared resources between threads, we need a control mechanism, namely a mutex. here we control access to `stdout`
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void shared_print(string origin, int value){
  mtx.lock();
  std::cout << origin << value << std:: endl;
  mtx.unlock();
}

void thr_func(){
  for (int i=0; i<100 ; i++){
    shared_print("t1 ", i);
  }
}
int main(){
  std::thread t1(thr_func);
  for(int i=0; i<100; i++){
    shared_print("main ", i);
  }
  t1.join();
  return 0;
}
```
this code is not _thread-safe_. if the resource access line produces an exception, we will have a locked out mutex. the standard library offers some alternatives.

### 12.1. lock_guard
```cpp
void shared_print(string origin, int value){
  std::lock_guard<std::mutex> guard(mtx);
  std::cout << origin << value << std:: endl;
}
```
as soon as the guard is defined, the mutex will be locked. it's unlocked when the guard goes out of scope. this declaration however does not prevent other threads from using `stdout`. <br>
typically thread-safe resource management is done with a class where the resource is a private member and all the functions that access the resource, control access with a mutex. such classes have the following properties :
* use mutex to synchronize resource access
* never return the resource handle
* never pass the resource handle as an argument to user provided functions
* design resource interface appropriately
### 12.2. multiple mutexes
assuming we have multiple resources and use multiple mutexes to control access, it's very possible to reach a deadlock
```cpp
void shared_access_1(string origin, int value){
  std::lock_guard<std::mutex> guard(mtx_1);
  std::lock_guard<std::mutex> guard(mtx_2);
  // access resources
}
void shared_access_2(string origin, int value){
  std::lock_guard<std::mutex> guard(mtx_2);
  std::lock_guard<std::mutex> guard(mtx_1);
  // access resources
}
```
#### solution 1
always lock the mutexes in the same order
#### solution 2
we can simultaneously lock multiple mutexes using `std::lock`
```cpp
void shared_access(string origin, int value){
  std::lock(mtx_1, mtx_2);
  std::lock_guard<std::mutex> guard(mtx_1, std::adopt_lock);
  std::lock_guard<std::mutex> guard(mtx_2, std::adopt_lock);
  // access resources
}
```
`std::adopt_lock` tells the guards that the mutex is already locked, but it should be unlocked when the guard goes out of scope.
#### solution 3
we can control the guards using arbitrary scopes
```cpp
void shared_access(string origin, int value){
  {
    std::lock_guard<std::mutex> guard(mtx_1, std::adopt_lock);
    // access resource 1
  }
  {
    std::lock_guard<std::mutex> guard(mtx_2, std::adopt_lock);
    // access resources
  }
}
```
### 12.3. unique lock
`unique_lock` provides more flexibility w.r.t `lock_guard` however it's at the cost of performance.
```cpp
void shared_access(string origin, int value){
  std::unique_lock<std::mutex> u_lock(mtx);
  // access resources
  ulock.unlock();
  // do something else
}
```
it's possible to unlock `unique_lock` before it goes out of scope. we can also defer the lock
```cpp
void shared_access(string origin, int value){
  std::unique_lock<std::mutex> u_lock(mtx, std::defer_lock);
  // do something else
  ulock.lock()
  // access resources
  ulock.unlock();
  // do something else
  ulock.lock()
  // access resources
  ulock.unlock();
}
```
we can also transfer ownership of `unique_lock`s using `std::move` as opposed to `lock_guard`s.
#### dealing with lazy initialization
when we initialize a resource for the first time when we want to access it, we have lazy initialization. in this case we'd need two mutexes, one for initialization and one for access.
```cpp
class FileAccess{
  std::mutex file_access_;
  std::mutex file_open_;
  
  void shared_write(string msg){
    std::unique_lock<mutex> ulock_open(file_open_);
    if(!file_.open()){
      file_.open("log.txt");
    }
    std::unique_lock<mutex> ulock_access(file_access_);
    file_.write(...);
    ulock_access.unlock();
  }
};
```
this program is now thread-safe but a lot of cpu cycles are wasted by waiting for the mutex just to check whether the already opened file is open. the standard library has a solution for this situation
```cpp
class FileAccess{
  std::mutex file_access_;
  std::once_flag open_flag_;
  
  void shared_write(string msg){
    std::call_once(open_flag_, [&](){ file_.open("log.txt"); });
    std::unique_lock<mutex> ulock_access(file_access_);
    file_.write(...);
    ulock_access.unlock(); // arbitrary
  }
};
```
`call_once` will execute the function (or callable object) passed to it only once across several threads.

### 12.4. sleeping and condition variable
```cpp
#include <deque>
#include <mutex>

std::deque<int> queue;
std::mutex mtx;

void function_1(){
  int count = 0;
  while (count<10){
    std::unique_lock<std::mutex> ulock(mtx);
    queue.push_front(count);
    ulock.unlock();
    std::this_thread::sleep_for(chrono::seconds(1));
    count++;
  }
}
void function_2(){
  int data=0;
  while (data!=9){
    std::unique_lock<std::mutex> ulock(mtx);
    if(!queue.empty()){
      data = queue.back();
      queue.pop_back();
      ulock.unlock();
      std::cout << "got " << data << " from first thread" << std::endl;
    }
    else{
      ulock.unlock();
      std::this_thread::sleep_for(chrono::milliseconds(10));
    }
  }
}
```
the first thread is creating data and the second thread is using it. if the second thread sleeps two shortly, it will use too many cpu cycles. if it sleeps for too long, we waste time. a solution to this problem is provided by the standard library. we declare a new global variable
```cpp
std::condition_variable cond;
void function_1(){
  ...
  ulock.unlock();
  cond.notify_one();
}
```
this will notify one (if any) thread that's waiting on this condition variable.
```cpp
void function_2(){
  while (data!=9){
    std::unique_lock<std::mutex> ulock(mtx);
    cond.wait(ulock);
    // access
    ulock.unlock();
  }
}
```
the wait function also takes the lock as a parameter to unlock the mutex while it's sleeping. when it wakes up, it will lock the mutex again.<br>
*it's never a good idea to lock a mutex and go to sleep*<br>
since we lock/unlock the mutex too many times, we can only use `unique_lock`. <br>
this setup would be fine if thread #2  would only wake up by the notification from thread #1, however it can wake up on its own, also called a *spurious wake*. to put the thread back to sleep after a spurious wake, we can pass a predicate as the second arguement
```cpp
cond.wait(ulock, [](){ return !queue.empty(); });
```
if we want every thread waiting on the condition variable to wake up, we can use `cond.notify_all()`.

### 12.5. future
if we want to get the results back from a child thread, we can share a variable between the child and the parent. however since both might access it, we also need a `mutex`. the parent should access the variable after the child, so we also need a `condtion_variable`. <br>
even for a simple task, we'd need to have 2 global variables and take care of locking and unlocking the mutex and the condition variable. standard library provides an interface for such cases
```cpp
#include <future>

int funct(int param){
   // do processing
  return some_int;
}
int main(){
  int x;
  std::future<int> cmp = std::async(funct, 4);
  x = cmp.get();
  return 0;
}
```
the get function waits until the child thread is done. calling this function twice will cause a crash.
#### thread or no thread
the async function takes a parameter that decides whether a thread is made for the function call or not
```cpp
// creates a thread
std::future<int> cmp = std::async(std::launch::async, funct, 4);
// direct call in the main thread
std::future<int> cmp = std::async(std::launch::deferred, funct, 4);
// default value, decided based on the implementation
std::future<int> cmp = std::async(std::launch::async|std::launch::deferred, funct, 4);

```
### 12.6. promise
we used future to send a variable from the child thread to the parent. we can do the reverse with a promise
```cpp
#include <future>

int funct(std::future<int> &ftr){
  ...
  int param = ftr.get();
  // do processing
  return some_int;
}
int main(){
  int x;
  std::promise<int> prm;
  std::future<int> ftr = p.get_future();
  std::future<int> cmp = std::async(std::launch::async, funct, std::ref(ftr));
  std::this_thread::sleep_for(chrono::milliseconds(20));
  // keeping the promise
  prm.set_value(4);
  x = cmp.get();
  return 0;
}
```
if we do not keep our promise (and also don't wait for the return variable), we'll get an exception of type `std::future_error::broken_promise`. if something is preventing us from keeping the promise, we can set and exception instead
```cpp
prm.set_exception(std::make_exception_ptr(std::runtime("oops")));
```
futures and promises cannot be copied, they can only be moved.
#### many promises
if our function has to be executed e.g. 10 times, we cannot pass the same `std::future` & `std::promise` to them. one way is to create 10 of each. standard library also provides shared future
```cpp
int funct(std::shared_future<int> sftr){
  ...
}
int main(){
  int x;
  std::promise<int> prm;
  std::future<int> ftr = p.get_future();
  std::shared_future sftr = ftr.share();
  std::future<int> cmp = std::async(std::launch::async, funct, sftr);
  std::future<int> cmp = std::async(std::launch::async, funct, sftr);
  std::future<int> cmp = std::async(std::launch::async, funct, sftr);  
  ...
  prm.set_value(4);
  ...
}
```
a `shared_future` can be copied which means we can also send it to the threads by value.
### 12.7. callable objects
the standard library has a uniform interface when it comes to callable objects. e.g. we have a functor A
```cpp
class A {
  void func(int x, char c) {}
  int operator()(int N){ return 0; }
};
void foo(int x) {}
int main(){
  A a;
  std::thread t1(a, 6);
  std::async(std::launch::async, a, 6);
  std::bind(a, 6);
  std::call_once(once_flag, a, 6);
  return 0;
}
```
#### ways to call them
```cpp 
int main(){
  A a;
  std::thread t1(a, 6);  		// copy of a() as functor in a different thread
  std::thread t2(std::ref(a), 6);	// a() as functor in a different thread
  std::thread t2(std::move(a), 6);	// a() as functor moved to a different thread
  std::thread t4(A(), 6);		// temporary A moved to a different thread
  std::thread t5([](int x){ return x*x;}, 6);
  std::thread t6(foo, 7);
  std::thread t7(&A::f, a, 8, 'w');	// copy of a.func(8, 'w') in a different thread
  std::thread t8(&A::f, &a, 8, 'w');	// a.f(8, 'w') in a different thread
}
```
### 12.8. packaged tasks
tasks that can be package and sent to different functions, objects or threads. tasks can be executed in different contexts from where they were created.
```cpp
int funct(int N){
  // do processing
  return some_int;
}
int main(){
  std::packaged_task<int(int)> task(funct);
  // some stuff
  task(4);
  int ret_value = task.get_future().get();
  return 0;
}
```
we cannot create tasks with parameters as we would do with threads. to do this we have to use `std::bind` to bind our function with the parameters that we want, returning a function object. 
```cpp
std::packaged_task<int()> task(std::bind(funct, 10));
// some stuff
task();
```
note that this new function object cannot take a parameter anymore because it's already bundled with the function.
#### why not just use the function object?
```cpp
auto func_obj = std::bind(func, 10);
// some stuff
func_obj();
```
a `packaged_task` can link a callable object to a `future` which is very useful in threading. e.g. here we push the task in a queue so that another thread can execute it.
```cpp
std::deque<std::packaged_task<int()>> task_queue;
std::mutex mtx;

void thread_func(){
  std::packaged_task<int()> task;
  {
    std::unique_lock<std::mutex> ulock(mtx);
    cond.wait(ulock, []() { return !task_queue.empty(); });
    task = std::move(task_queue.front());
    task_queue.pop_front();
  }
  task();
}

int main(){
  std::thread thread(thread_func);
  std::packaged_task<int()> task(std::bind(funct, 10));
  std::future<int> ftr = task.get_future();
  {
    std::lock_guard<std::mutex> lock(mtx);
    task_queue.push_back(std::move(task));
  }
  cond.notify_one();
  std::cout << "output of a task, created in main, executed in another thread, received in main "<< ftr.get();
  thr.join();
  return 0;
}
```
thread-safe and useless.
### 12.9. time constraints
#### thread
```cpp
std::this_thread::sleep_for(chrono::milliseconds(3));
chrono::steady_clock::time_point tp = chrono::steady_clock::now() + chorno::microseconds(4);
std::this_thread::sleep_until(tp);
```
#### mutex
```cpp
std::unique_lock<std::mutex> ulock(mtx, std::defer_lock);
ulock.try_lock();
ulock.try_lock_for(chrono::nanoseconds(500));
ulock.try_lock_until(tp); 
```
`try_lock` returns if it cannot lock the mutex.
#### condition variable
```cpp
std::condition_variable cond;
cond.wait_for(ulock, chrono::microseconds(2));
cond.wait_until(ulock, tp);
```
#### future
```cpp
std::future<int> ftr;
ftr.wait();
ftr.wait_for(chorno::seconds(10));
ftr.wait_until(tp);
```
the wait function waits until the output of future is read.`ftr.get()` internally calls the wait function.
## 13. OpenCV
uses own types
#### naming convention
types follow the patern `CV_<bit_count><identifier><num_of_channels>` e.g. `CV_8UC3` is 8-bit unsigned char with 3 channels for RGB. `CV_8UC1` is 9-bit unsigned char grayscale. 

it's better to use `DataType`. e.g. `DataType<uint>::type == CV_8UC1`
### 13.1. basic matrix type
#### constructors
```cpp
cv::Mat image(rows, cols, DataType, value);
cv::Mat_<T> image(rows, cols, value);
```
#### initialization
```
cv::Mat iamge = cv::Mat::zeros(10, 10, CV_8UC3);
using Matf = cv::Mat_<float>;
Matf image = Matf::zeros(10, 10);
```
#### properties
get type using `image.type()`<br>
get size with `image.rows`, `image.cols`

### 13.2. memory management
`cv::Mat` is a shared pointer, although not a `std::shared_ptr`. cloning can be done :
```cpp
cv::Mat image = cv::Mat::zeros(10,10);
cv::Mat nocopy = image;
cv::Mat copy = image.clone();
```
#### command line compiling
```bash
c++ -std=c++11 -o main main.cpp `pkg-config --libs --cflags opencv`
```

### 13.3. IO
#### reading
```cpp
Mat imread(const string& file, int mode=1)
```
modes<br>
* unchanged   `CV_LOAD_IMAGE_UNCHANGED < 0`
* 1 channel   `CV_LOAD_IMAGE_GRAYSCALE == 0`
* 3 channels  `CV_LOAD_IMAGE_COLOR > 0`
```cpp
Mat i1 = imread("logo.png", CV_LOAD_IMAGE_GRAYSCALE);
Mat_<unit8_t> i2 = imread("logo.png", CV_LOAD_IMAGE_GRAYSCALE);
std::out << (i1.type == i2.type) << std::endl;
```
it's recommended to use typed matrices. This will help when we're assinging values to individual pixels.
#### writing
```cpp
bool imwrite(const string& file, const Mat& img)
```
simple type conversion
```cpp
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
int main(){
  cv::Mat image = imread("logo.png", CV_LOAD_IMAGE_COLOR);
  cv::imwrite("copy.jpg", image);
  return 0;
}
```
#### exr files
when stroing floating point images, OpenCV expects the values to be in 0-1 range. when storing arbitrary values, there might be cutoffs. to avoid this, we can save to `.exr` files. they will store and read the values as is without losing precision.

#### showing
```cpp
void imshow(const string& window_name, const Mat& mat);
```
use 
```cpp
#include <opencv2/opencv.hpp>
int main(){
  cv::Mat mat = cv::imread(...);
  cv::namedWindow("window", cv::WINDOW_AUTOSIZE);
  cv::imshow("window", mat);
  cv::waitKey();
}
```
### 13.4. vector type
```cpp
cv::Vec<Type, SIZE>
```
many typedefs available, e.g. `Vec3f`, `Vec3b`, etc. used ofr pixels in multidimensional images :
```cpp
cv::Mat mat = cv::Mat::zeros(10, 10, CV_8UC3);
std::cout << mat.at<Vec3b>(5, 4);

cv::Mat_<Vec3f> matf = cv::Mat_<Vec3f>::zeros(10, 10);
std::cout << matf.at<Vec3f>(1, 5);
```
#### using wrong types
will not produce bugs, but it will generate unwanted behavior. 

### 13.5. SIFT descriptors
we have `SiftFeatureDetector` to detect the keypoints and `SiftDescriptorExtractor` to compute the descriptors.
```cpp
// detect keypoints
SiftFeatureDetector detector;
vector<KeyPoint> keypoints;
decetor.detect(input, keypoints);
// show keypoints
cv::Mat image_with_keypoints;
drawKeypoints(input, keypoints, image_with_keypoitns);
// extract SIFT descriptors
SiftDescriptorExtractor extractor;
extractor.comput(input, keypoints, descriptors);
```
### 13.6. FLANN
Fast library for Aproximate Nearest Neighbors. builds K-d tree, searches for neighbors there.
```cpp
// create a kdtree for searching the data
cv::flann::KDTreeIndexParams index_params;
cv::flann::Index kdtree(data, index_params);
// search the nearest vector to some query
int k = 1;
cv::Mat nearerst_vector_idx(1, k, DataType<int>::type);
cv::Mat nearerst_vector_dist(1, k, DataType<float>::type);
kdtree.knnSearch(query, nearest_vector_idx, nearest_vector_dist, k);
```

## 14. bind
it "binds" parameters to callables and returns a new callable. usually `auto` is used to catch the return type but it can be cast to a `std::function` to pass it around. for better or worse callable objects returned by bind swallow all the extra parameters. bind has a bit of an overhead and in most cases can be replaced with lambdas.

```cpp
#include <functional>
#include <iostream>

void Print(int i) {
	std::cout << i << '\n';
}
int main()
{
	int i = 5;
	auto f = std::bind(&Print, i);
	f();
}
```
### 14.1. binding with reference
```cpp
void Print(const int& i) {
	std::cout << i << '\n';
}
int main()
{
	int i = 5;
	auto f = std::bind(&Print, i);
	f();	// 5
	i = 6;
	f();	// 5
	f = std::bind(&Print, std::ref(i));
	f();	// 5
	i = 6;
	f();	// 6
}
```
### 14.2. binding arbitrary arguments
```cpp
void Print(const int& i, const std::string& str) {
	std::cout << i << ' ' << str << '\n';
}
int main()
{
	int i = 5;
	auto f = std::bind(&Print, std::ref(i), std::placeholders::_1);
	f("hi");
}
```
### 14.3. binding with arbitrary order
```cpp
void Print(const int& i, const std::string& str) {
	std::cout << i << ' ' << str << '\n';
}
int main()
{
	int i = 5;
	auto f = std::bind(&Print, std::placeholders::_2, std::placeholders::_1);
	f("hi", 20);
}
```
### 14.4. binding with template functions
```cpp
template<typename T>
void Print(T i, const std::string& str) {
	std::cout << i << ' ' << str << '\n';
}
int main()
{
	int i = 5;
	auto f = std::bind(&Print, std::placeholders::_2, std::placeholders::_1); 		// fails
	auto f = std::bind(&Print<int>, std::placeholders::_2, std::placeholders::_1);	// works
	f("hi", 20);
}
```
### 14.5. bind sucks
same thing can be done with a lambda, maybe even with more flexibility. the variadic arg is added to swallow extra args just like bind would. we could also add another argument to be swallowed in the front by adding `auto &&`. doing the same thing with bind is up to 20x slower. it also removes the need to specifiy templates during declaration.
```cpp
template<typename T>
void Print(T i, const std::string& str) {
    std::cout << i << ' ' << str << '\n';
}
int main()
{
	int i = 5;
	const auto f = [] (const std::string &arg1, auto &&arg2, auto &&...) {
		print(arg2, arg1);
	};
	f("hello", i, 5);
	i = 6;
	f("hello", i, 5, 2, 1.2f);
}
```
## 15. C++14
### 15.1. exchange
uses move semantics to update a variable and it's history variable efficiently without making any copies. typical use case:
```cpp
#include <list>
#include <algorithm>
#include <iostream>
int main() 
{
	int var, var_last;
	while (condition_not_met) {
		// inefficient
		var_last = var;
		var = new_value;
		// efficient
		var_last = std::exchange(var, new_value);
	}
}
```
## 16. C++17
### 16.1. structured bindings
when dealing with multiple return types we had several options
```cpp
std::tuple<std::string, int> CreatePerson()
{
  return {"cherno", 48};
}
```
#### struct
this is the old fashioned way. we'd create a struct containing both types, instead of using tuple.
#### std::get
```cpp
auto person = CreatePerson();
std::string name = std::get<0>(person);
int age = std::get<1>(person);
```
#### std::tie
```cpp
std::string name;
int age;
std::tie(name, age) = CreatePerson();
```
#### structured binding
```cpp
auto[name, age] = CreatePerson();
```

### 16.2. any
an experimental feature of C++17, it's a type-safe container that can contain any single object:
```cpp
#include <experimental/any>
#include <vector>
#include <string>

int main()
{
	std::vector<std::experimental::fundamentals_v1::any> vec(1, 3.4f, 4.01, std::string("hWs"));
	std::cout << vec.size() << '\n';
	std::cout << std::experimental::fundamentals_v1::any_cast<int>(v[0]) << '`\n';
	std::cout << std::experimental::fundamentals_v1::any_cast<float>(v[0]) << '`\n'; // throws bad any_cast exception
	std::cout << vec[1].type().name() << '`\n';
}
```
the type might require demangling as explained in the `typeid` section. for on-the-fly demangling : `./a.out | c++-filt -t`

#### copy-constructible
all objects pushed into std::any must be copy constructible. e.g. this wouldn't compile:
```cpp
struct S {
	S(const S &s) = delete; // explicitly deleting copy constructor (implicitely deletes default constructors)
	S() = default;          // explicitly redefining default constructor
};
int main()
{
	std::vector<std::experimental::fundamentals_v1::any> vec(1, 3.4f, 4.01, std::string("hWs"), S());
}

```
implementations are encouraged to avoid dynamic allocations as long as the object is small and `std::is_nothrow_move_constructible`.
#### copies and references
the bracket operator doesn't return a reference as expected. we can only get points :
```cpp
int main()
{
	std::vector<std::experimental::fundamentals_v1::any> vec(1, 2);
	int &i = std::experimental::fundamentals_v1::any_cast<int>(v[0]);   // fails
	int &i = std::experimental::fundamentals_v1::any_cast<int&>(v[0]);  // fails
	int *i = std::experimental::fundamentals_v1::any_cast<int>(&v[0]);  // works
}
```
### 16.3. if and switch initialization
it's possible to initialize the expression that is used in an if or switch statement. it lives on for the entire scope of if/else or switch. it's simillar to having opening/closing braces around the entire scope.
```cpp
#include <vector>

int main()
{
	std::vector<int> vec{1,2,3,4};
	if (auto itr = vec.find(2);
			itr != vec.end()) {
		*itr = 5;
	} else {
		vec.emplace_back(2);
	}
}
```
### 16.4. nested namespaces
to avoid having multiple levels braces for nested namespaces, we can directly use the nested namespace:
```cpp
namespace torch::tensor::float
{

}
```
### 16.5. has include
this macro gives us the ability to check the availability of header files
```cpp
// before C++17
#if defined(__linux__) || defined(__unix__) || defined(__APPLE__) || defined(__HAIKU)
#include <unistd.h>
#endif
#if defined(_WIN32)
#include <windows.h>
#endif
// after C++17
#if __has_include(<unistd.h>)
#include <unistd.h>
#endif
#if __has_include(<windows.h>)
#include <windows.h>
#endif
```
### 16.6. aggregate initialization
C++11 allowed uniform intializations even in the absence of a constructor using `{}`. this however runs into problems if the object we're initializing is from a child class.
```cpp
struct Base {
  double d;
};
struct Child : Base {
  int i;
  float f;
};
int main()
{
  Child ch_1{1, 2.4f};      // error: implicit copy constructor requires 1 argument
                            //        implicit move constructor requires 1 argument
                            //        implicit default constructor requires 0 arguments
  Child ch_2{{}, 1, 2.4f};  // using default intialization for q from base class
  Child ch_3{{15.1}, 1, 2.4f};

}
```
### 16.7. deduction guides
we can help the compiler deduce types where it's confused. it has to be in the same namespace:
```cpp
#include <functional>

namespace std // same namespace has std::function
{
  // deduce an std::function from a standalone function pointer
  template<typename ReturnType, typename ...Args> function(Ret (*)Args...) -> function<ReturnType(Args..)>;
  // deduce an std::function standalone from a member function pointer
  template<typename ReturnType, typename Class, typename ...Args> function(ReturnType(Class::*)(Args...)) 
    -> function<ReturnType(Class&, Args..)>;
  // deduce an std::function standalone from a const member function pointer
  template<typename ReturnType, typename Class, typename ...Args> function(ReturnType(Class::*)(Args...) const) 
    -> function<ReturnType(const Class&, Args..)>;
}
struct MyClass {
  void member_function();
  void const_member_function() const;
}
void standalone_function(int, char) {

};
int main()
{
  std::function sa_f(&standalone_function);
  sa_f(1, 'c');
  std::function m_f(&MyClass::member_function);
  std::function cm_f(&MyClass::const_member_function);
}
```
## 17. C++20

## 18. Attributes
any code that contains attributes is automatically c++98 incompatible but who cares.
### 18.1. fallthrough
`[[C++17]]` in switch statements we might need to only slightly differentiate between two cases, i.e. do a bunch of stuff for case A and proceed with what we would've done for case B. if they're identical, we won't have any problems but otherwise the compiler issues a fallthrough warning.
```cpp
switch(expression) {
  case 1:
    do_some_stuff();
    [[fallthrough]];
  case 2:
    do_other_stuff;
    break;
}
```
### 18.2. unused
`[[C++17]]` there are cases where it's possible to have a local variable that is technically unused but not needed. e.g. :
```cpp
int main()
{
  [[unused]] int i = 6;
  assert(i == 6);
}
```
### 18.3. maybe_unused
`[[C++17]]` there are a couple of ways to deal with unused arguements in a function.
* don't give it a name
* comment out the name
* `[[maybe_unused]]`
```cpp
int main(int /*argc*/, [[maybe_unused]] const char** argv)
{

}
```
functions can also be `[[maybe_unused]]`:
```cpp
[[maybe_unused]] void do_nothing() {

}
```
### 18.4. nodiscard
`[[C++17]]` one difference between returned error codes & exceptions is that error codes can be ignored. it is possible to enforce a check and ignoring it requires a lot more effort.
```cpp
[[nodiscard]] int read_file(const char* name, char* buf, int length) {

}
int main()
{
  char *buff[200];
  [[maybe_unused]] int ret = read_file("text.txt", buff, 200);
}
```
it can also be applied to structs, classes, enums, unions, etc. 
## 99. Misc.
### 99.1. timing
`chrono` is a platform independent mechanism for timing introduced to the standard library after C++11. <br>
the following code measures duration in seconds
```cpp
#include <chrono>
#include <iostream>
#include <thread>

int main() {
  using namespace std::literals::chrono_literals;
  auto start = std::chrono::high_resolution_clock::now();
  std::this_thread:sleep(1s);
  auto end = std::chrono::high_resolution_clock::now();
  std::chrono::duration<float>  duration = end - start;
  std::cout << duration.count() << "s" << std::endl;
  return 0;
}
```
for easier use of sleep functions we can use `chrono_literals`.
#### structured timing
```cpp
struct Timer {
  std::chrono::time_point<std::chrono::steady_clock> start, end;
  std::chrono::duration<float>  duration;
  Timer(){
    start = std::chrono::high_resolution_clock::now();
  }
  ~Timer(){
    end = std::chrono::high_resolution_clock::now();
    duration = end-start;
    float ms = duration.count()*1000.0f;
    std::cout << "chrono timer " << ms << "ms" << std::endl;
  }
}
void Function() {
  Timer timer;
  for(int i=0; i<100; ++i){
    std::cout << "hello" << std::endl;
  }
}
```
now any function that declares a `Timer` object will be timed.

### 99.2. variadic templates
In C++11 there are two new options, as the Variadic functions reference page in the Alternatives section states:
* Variadic templates can also be used to create functions that take variable number of arguments. They are often the better choice because they do not impose restrictions on the types of the arguments, do not perform integral and floating-point promotions, and are type safe. (since C++11)
* If all variable arguments share a common type, a std::initializer_list provides a convenient mechanism (albeit with a different syntax) for accessing variable arguments.
```cpp
#include <iostream>
#include <string>
#include <initializer_list>

template <typename T>
void func(T t) 
{
    std::cout << t << std::endl ;
}

template<typename T, typename... Args>
void func(T t, Args... args) // recursive variadic function
{
    std::cout << __PRETTY_FUNCTION__ << ": " << t <<std::endl ;
    func(args...) ;
}

template <class T>
void func2(std::initializer_list<T> list )
{
    for( auto elem : list )
    {
        std::cout << __PRETTY_FUNCTION__ << ": " << t <<std::endl ;
    }
}

int main()
{
    std::string
    str1( "Hello" ),
    str2( "world" );

    func(1,2.5,'a',str1);

    func2( {10, 20, 30, 40 }) ;
    func2( {str1, str2 } ) ;
}
```
the `__PRETTY_FUNCTION__` directive only works with gcc and clang. the output would be 
``` cpp
void func(T, Args...) [T = int, Args = <double, char, std::basic_string<char>>]: 1
void func(T, Args...) [T = double, Args = <char, std::basic_string<char>>]: 2.5
void func(T, Args...) [T = char, Args = <std::basic_string<char>>]: a
void func(T) [T = std::basic_string<char>]: Hello
```



### 99.3. next
gets the next iterator. for vector, we can directly increment the iterator because they're random access. list on the other hand isn't.

```cpp
#include <list>
#include <algorithm>
#include <iostream>
int main() 
{
	std::list<int> lst(10, 1, 2, 3, 4, 5);
	std::cout << std::is_sorted(std::begin(lst), std::end(lst)) << '\n';        // false
	std::cout << std::is_sorted(std::next(begin(lst)), std::end(lst)) << '\n';  // true
}
```
