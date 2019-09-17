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
6. [Move Semantics](#6-move-semantics)<br>
  6.1. [lvalue and rvalue](#61-lvalue-and-rvalue)
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
  10.4. [recursive lambdas](#104-recursive-lambdas)
11. [OpenCV](#11-opencv)<br>
  11.1. [basic matrix type](#111-basic-matrix-type)<br>
  11.2. [memory management](#112-memory-management)<br>
  11.3. [IO](#113-io)<br>
  11.4. [vector type](#114-vector-type)<br>
  11.5. [SIFT descriptors](#115-sift-descriptors)<br>
  11.6. [FLANN](#116-flann)
  
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
by c++ standard, after an object has been moved, it should be empty. moving a variable transfer ownership of its resource to another variable. runtime is better than copying, worse than passing by reference.
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
## 11. OpenCV
uses own types
#### naming convention
types follow the patern `CV_<bit_count><identifier><num_of_channels>` e.g. `CV_8UC3` is 8-bit unsigned char with 3 channels for RGB. `CV_8UC1` is 9-bit unsigned char grayscale. 

it's better to use `DataType`. e.g. `DataType<uint>::type == CV_8UC1`
### 11.1. basic matrix type
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

### 11.2. memory management
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

### 11.3. IO
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
### 11.4. vector type
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

### 11.5. SIFT descriptors
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
### 11.6. FLANN
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
