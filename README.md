# C++ Reference ([Part 2](modern.md))
1. [Types and Stuff](#1-types-and-stuff)<br>
  1.1. [decltype and auto](#11-decltype-and-auto)<br>
  1.2. [declval](#12-declval)<br>
  1.3. [typedef and using](#13-typedef-and-using)<br>
  1.4. [unions](#14-unions)<br>
  1.5. [enums](#15-enums)<br>
  1.6. [volatile keyword](#16-volatile-keyword)<br>
  1.7. [static keyword](#17-static-keyword)<br>
  1.8. [mutable keyword](#18-mutable-keyword)<br>
  1.9. [extern keyword](#19-extern-keyword)<br>
  1.10. [typename keyword](#110-typename-keyword)<br>
  1.11. [storage classes](#111-storage-classes)<br>
2. [Raw Pointers](#2-raw-pointers)<br>
  2.1. [const keyword](#21-const-keyword)<br>
  2.2. [function pointers](#22-function-pointers)<br>
3. [Class Basics](#3-class-basics)<br>
  3.1. [other ways of instantiating](#31-other-ways-of-instantiating)<br>
  3.2. [member initialization](#32-member-initialization)<br>
  3.3. [static members](#33-static-members)<br>
  3.4. [const objects and functions](#34-const-objects-and-functions)<br>
  3.5. [template specialization](#35-template-specialization)<br>
  3.6. [template instantiation](#36-template-instantiation)<br>
  3.7. [extern templates](#37-extern-templates)
4. [Inheritance](#4-inheritance)<br>
  4.1. [friend functions and classes](#41-friend-functions-and-classes)<br>
  4.2. [access specifiers](#42-access-specifiers)<br>
  4.3. [what's inherited](#43-whats-inherited)<br>
  4.4. [base constructor](#44-base-constructor)<br>
  4.5. [virtual functions](#45-virtual-functions)<br>
  4.6. [pure virtual and abstract classes](#46-pure-virtual-and-abstract-classes)<br>
  4.7. [virtual destructors](#47-virtual-destructors)<br>
  4.8. [virtual inheritance](#48-virtual-inheritance)<br>
  4.9. [virtual tables and dynamic dispatch](#49-virtual-tables-and-dynamic-dispatch)<br>
  4.10. [type erasure](#410-type-erasure)
5. [Type Conversion](#5-type-conversion)<br>
  5.1. [through classes](#51-through-classes)<br>
  5.2. [explicit keyword](#52-explicit-keyword)<br>
  5.3. [type casting](#53-type-casting)<br>
  5.4. [typeid](#54-typeid)<br>
  5.5. [boolean conversions](#55-boolean-conversions)<br>
  5.5. [integer conversions](#56-integer-conversions)<br>
  5.7. [implicit arithmetic conversions](#57-implicit-arithmetic-conversions)<br>
6. [Exception](#6-exception)<br>
  6.1.[specifcation](#61-specification)<br>
  6.2.[standard exceptions](#62-standard-exceptions)
7. [File IO](#7-file-io)<br>
  7.1. [modes](#71-modes)<br>
  7.2. [ascii reading](#72-ascii-reading)<br>
  7.3. [ascii writing](#73-ascii-writing)<br>
  7.4. [binary reading](#74-binary-reading)<br>
  7.5. [binary writing](#75-binary-writing)
8. [Functors](#8-functors)<br>
  8.1. [pros](#81-pros)<br>
  8.2. [runtime templates](#82-runtime-templates)<br>
  8.3. [built-in functors](#83-built-in-functors)<br>
  8.4. [parameter binding](#84-parameter-binding)<br>
  8.5. [regular function to functor](#85-regular-function-to-functor)<br>
  8.6. [getting more complex](#86-getting-more-complex)<br>
  8.7. [functors and sorting](#87-functors-and-sorting)<br>
  8.8. [predicates](#88-predicates)<br>
  8.9. [template functors](#89-template-functors)
9. [Iterators](#9-iterators)<br>
  9.1. [begin and end](#91-begin-and-end)<br>
  9.2. [increment and decrement operators](#92-increment-and-decrement-operators)<br>
  9.3. [index operator](#93-index-operator)<br>
  9.4. [arrow operator](#94-arrow-operator)<br>
  9.5. [dereference operator](#95-dereference-operator)<br>
  9.6. [comparison operators](#96-comparison-operators)<br>
10. [Precompiled Headers](#10-precompiled-headers)<br>
11. [IEEE 754 and caveats](#11-ieee-754-and-caveats)<br>
  11.1. [special floats](#111-special-floats)<br>
  11.2. [exponent encoding](#112-exponent-encoding)<br>
  11.3. [epsilon](#113-epsilon)<br>
  11.4. [significant digits](#114-significant-digits)<br>
  11.5. [denormalized numbers](#115-denormalized-numbers)<br>
  11.6. [precision](#116-precision)<br>
  11.7. [units in last place](#117-units-in-last-place)<br>
  11.8. [relative error](#118-relative-error)<br>
  11.9. [rounding error](#119-rounding-error)<br>
  11.10. [extra bits](#1110-extra-bits)<br>
  11.11. [extra algebraic assumption error](#1111-algebraic-assumption-error)<br>
  11.12. [floating point exceptions](#1112-floating-point-exceptions)<br>
  11.12. [float tricks](#1113-float-tricks)<br>

# 1. Types and Stuff
## 1.1. decltype and auto
`auto` deduces the type (obviously) at compile time.
```cpp
int a = 5;
decltype(a) b;  // type of a without initialization
auto c = a;     // type of a with initialization
```
`auto` has the same rules for `const`-ness, references and pointers as templates:
* `auto` will never deduce a reference
* `const`-ness will only be deduced for references and pointers
* value types are always copied (special case for `decltype`)

```c++
const int *p = nullptr;
const int i = 0;

auto a1 = p;           // const int*
auto *a2 = p;          // const int*
auto a3 = *p;          // int
auto &a5 = *p;         // const int&
auto a5 = i;           // int
decltype(auto) a6 = i; // const int
```
auto also _never_ performs a conversion, making it sometimes better than writing down the type explicitly, although it might perform an unwanted copy. `decltype(auto)` can be used for variable declaration, function return type and trailing return type with the purpose of removing the inherent limitations of `auto`, namely that it never deduces references and constness (except for explicit const references).
## 1.2. declval
`std::declval` converts any type T to a reference type, making it possible to use member functions in decltype expressions without the need to go through constructors.
```cpp
struct Default { int foo() const { return 1; } };
 
struct NonDefault
{
    NonDefault() = delete;
    int foo() const { return 1; }
};
 
int main()
{
    decltype(Default().foo()) n1 = 1;                   // type of n1 is int
//  decltype(NonDefault().foo()) n2 = n1;               // error: no default constructor
    decltype(std::declval<NonDefault>().foo()) n2 = n1; // type of n2 is int
    std::cout << "n1 = " << n1 << '\n'
              << "n2 = " << n2 << '\n';
}
```
## 1.3. typedef and using
declaring new types :
```cpp
typdef char C;
using C = char;
```
### type aliasing
`using` can be used in combination with classes, temlate types and functions.
```cpp
// inside classes
template <class T, int SIZE>
class Image
{
    using Ptr = std::unique_ptr<Image<T, SIZE>>;
};
// with template
template <int SIZE>
using Imagef = Image<float, SIZE>;

// inside functions
int main()
{
    using Image3f = Imagef<3>;
    auto image_ptr = Image3f::Ptr(new Image3f);
}
```
## 1.4. unions
```cpp
union mix_t
{
    int l;
    struct {
        short hi;
        short lo;
    }s;
    char c[4];
}mix;
```
union trivia:
- a union always has the size of its largest member
- unions can have constructors and destructors since C++11
  - there is no way of knowing whether a union member was constructed
  - it is therefor impossible to get the destructor right using information just within the union :) proposal pending
- unions can also have member functions and operator overloading
- a union can have at most one active member at a time. there is no default member
  - accessing a non-active member is undefined behavior
- wrapping an object inside a (nameless) union can delay its initialization, e.g. `union UT { std::string s; }`
  - `s` will remain inactive until it is constructed, which is "impossibe" the normal way
  - such objects can be constructed using placement new: `new (&ut.s) std::string();`
  - or since C++23: `std::construct_at(&ut.s);`
## 1.5. enums
```cpp
//numbered starting from 0
enum colors_t {BLACK, BLUE, GREEN, CYAN, RED, PURPLE, YELLOW, WHITE};
//numbered startin from 1
enum months_t { JANUARY=1, FEBRUARY, MARCH, ...};
```
### enum class, enum struct
enum class or enum struct helps avoid dumb misakes by preventing unwanted int casts.
```cpp
enum class Colors {BLACK, BLUE, GREEN, CYAN, RED, PURPLE, YELLOW, WHITE};
Colors mycolor;
mycolor = Colors::BLUE;
if (mycolor == Colors::GREEN) mycolor = Colors::RED;
```

### determine type size
```cpp
enum class EyeColor : char {blue, green, brown};
```
### explicit values
usable for masks
```cpp
enum class EnumType {
  OPTION_1 = 10,
  OPTION_2 = 0xBE,
  ...
}
```
## 1.6. volatile keyword
volatile is a hint to the implementation to avoid aggressive optimization involving the object because the value of the object might be changed by means undetectable by an implementation. the compiler might sometimes optimize objects that affect program flow e.g. a loop. from the compiler's perspective the object could be changed by a static value but that might not be the case. to avoid such faults, the volatile keyword should be used.
## 1.7. static keyword
```cpp
int f(void) {
    static int x = 0;
    x++;
    return x;
}
int main(void)
{
    int j;
    for (j = 0; j < 5; ++j) {
        printf("value of f(): %d\n", f());
    }
    return 0;
}
```
returns 0 1 2 3 4<br>
static member functions of a class can be accessed without an object using the scope operator `::`. they can only access static members of the class and don't have access to `this` pointer. <br>
static functions outside classes are only declared in the scope of their respective files. they cannot be used outside that file, even when explicitly mentioned in a header file as a prototype.
## 1.8. mutable keyword
The mutable storage class specifier is used only on a class data member to make it modifiable even though the member is part of an object declared as const. You cannot use the mutable specifier with names declared as static or const, or reference members.
```cpp
class A
{
public:
    A() : x(4), y(5) {};
    mutable int x;
    int y;
};
int main()
{
    const A var2;
    var2.x = 345;
    var2.y = 2345; // not allowed
}
```
## 1.9. extern keyword
used to specify the type and the existance of an object. once all of the source files have been compiled, the linker will resolve all of the references to the one definition that it finds in one of the compiled source files. the definition of the object needs to have “external linkage”, which means that it needs to be declared outside of a function and without the static keyword.
```cpp
---header.h---
extern int gloabl_x;
void print_gloabl_x();
---source1.cpp---
#include "header.h"
int gloabl_x;
int main()
{
    gloabl_x = 5;
    print_global_x();
}
---source 2---
#include "header.h"
void print_gloabl_x() {
    std::cout << global_x;
}
```
## 1.10. typename keyword
`typename` is also used in contexts where the template resolution might confuse the compiler. For example if we want to create a pointer or use the value type of a templated type, it might be misconstrued as a multiplication wtih a member variable or a member variable on its own. Here we need the `typename` keyword to indicate that the identifier that follows is a type.

```cpp
template <typename T>
class Test {
    T::subtype *ptr; // Error: multiplying T::subtype member variable with something called ptr?
    typename T::subtype *ptr; // Ok

    T::subtype pop(); // Error: is the member variable T::subtype a return type?
    typename T::subtype pop(); // Ok
};
```
## 1.11. storage classes
the storage class specifiers are a part of the decl-specifier-seq of a name's declaration syntax. together with the scope of the name, they control two independent properties of the name: its storage duration and its linkage. 

* auto: automatic storage duration
* register: automatic storage duration. also hints to the compiler to place the object in the processor's register. removed in C++17
* static: static or thread storage duration and internal linkage
* extern: static or thread storage duration and external linkage
* thread_local: thread storage duration
* mutable - does not affect storage duration or linkage

# 2. Raw Pointers
## 2.1. const keyword
```cpp
int x;
      int *       p1 = &x;  // non-const pointer to non-const int
const int *       p2 = &x;  // non-const pointer to const int
      int * const p3 = &x;  // const pointer to non-const int
const int * const p4 = &x;  // const pointer to const int
const int * p2a = &x;  //      non-const pointer to const int
int const * p2b = &x;  // also non-const pointer to const int
```
### google style
CamelCase starting with `k`.
## 2.2. function pointers
### standalone functions
```cpp
int addition (int a, int b) { return a + b; }
int subtraction (int a, int b) { return a - b; }
int call (int x, int y, int (*functocall)(int, int))
{
    return (*functocall)(x, y);
}
int main ()
{
    int (*minus)(int,int) = subtraction;
    int m = call(7, 5, addition);
    int n = call(20, m, minus);
}
```
### member functions
```cpp
struct Num
{
    int a = 10;
    int add(int b) {
        return a + b;
    }
    int subtract(int b) {
        return a - b;
    }
};
int main()
{
    bool perform_add;
    std::cin >> perform_add;
    Num num_obj;
    int (Num::*fptr)(int) = nullptr;
    if (perform_add) {
        fptr = &Num::add;
    } else {
        fptr = &Num::subtract;
    }
    std::cout << (num_obj.*fptr)(20) << std::endl;
    // also possible with
    std::cout << std::invoke(&Num::add, num_obj, 13) << std::endl;
}
```

# 3. Class Basics
## 3.1. other ways of instantiating
```cpp
class Circle
{
    double radius;
    public:
    Circle(double r) { radius = r; }
};
Circle foo (10.0);   // functional form
Circle bar = 20.0;   // assignment init.
Circle baz {30.0};   // uniform init.
Circle qux = {40.0}; // POD-like
```
and also
```cpp
Rectangle rectb;   // default constructor called
Rectangle rectc(); // function declaration (default constructor NOT called)
Rectangle rectd{}; // default constructor called
```
in general, list initialization is preferred due to it's reluctance for narrowing. official reference:

list initialization does not allow narrowing (§iso.8.5.4). That is:

 * An integer cannot be converted to another integer that cannot hold its value. For example, char to int is allowed, but not int to char.
 * A floating-point value cannot be converted to another floating-point type that cannot hold its value. For example, float to double is allowed, but not double to float.
 * A floating-point value cannot be converted to an integer type.
 * An integer value cannot be converted to a floating-point type
## 3.2. member initialization
For members of fundamental types, it makes no difference how the constructor is defined, because they are not initialized by default, but for member objects (those whose type is a class), if they are not initialized using member initializer list, they are default-constructed:
```cpp
class A
{
    double r_;
public:
    A(double r) : r_(r) {}
};
class B
{
    A base;
public:
    Cylinder(double r) : base{r} {}
};
```
or just make a pointer to A like a decent person.
## 3.3. static members
### variables
exist exactly once per type, not per object. the value is equal across all instances.
```cpp
class Dummy
{
  static int counter;
  Dummy() {counter++;}
};
Dummy::counter = 0;
```
to avoid them being declared several times, they cannot be initialized directly in the class, but need to be initialized somewhere outside it (always in cpp files, never in headers).
### functions
they do not have an object of the class. they can access private members if they are passed to them as parameters. syntax:
```cpp
ClassName::MethodName(<params>)
```
## 3.4. const objects and functions
```cpp
const Class object;
```
* access to its data members from outside the class is restricted to read-only, as if all its data members were const for those accessing them from outside the class. member functions should be const if they are to be called outside the class.
* member functions specified to be const cannot modify non-static data members nor call other non-const member functions. In essence, const members shall not modify the state of an object.
* const objects are limited to access only member functions marked as const, but non-const objects are not restricted and thus can access both const and non-const member functions alike.
Most functions taking classes as parameters actually take them by const reference, and thus, these functions can only access their const members, i.e. ensuring that the passed object does not change. overloading constness is also a thing :
```cpp
class MyClass {
    int x;
public:
    MyClass(int val) : x(val) {}
    const int& get() const {
        return x;
    }
    int& get() {
        return x;
    }
};
```
## 3.5. template specialization
if the class uses templates but for a special data type, it should be declared a bit differently, we declare it twice. once with template classes, and once with an empty template and the specific data type:
```cpp
// class template:
template <class T>
class mycontainer {
T element;
public:
    mycontainer (T arg) {
        element = arg;
    }
    T increase () {
        return ++element;
    }
};
// class template specialization:
template <>
class mycontainer <char>
{
char element;
public:
    mycontainer(char arg) {
        element = arg;
    }
    char uppercase()
    {
        if (element>= 'a' && element <= 'z') {
            element += 'A' - 'a';
        }
        return element;
    }
};
```
notice that we precede the class name with `template<>` with an empty parameter list. this is because all types are known and no template arguments are required for this specialization, but still, it is the specialization of a class template, and thus it requires to be noted as such. when we declare specializations for a template class, we must also define all its members, even those identical to the generic template class, because there is no "inheritance" of members from the generic template to the specialization.
## 3.6. template instantiation
explicit instantiation can be used when the templated type or function is declared inside a header and defined inside a source file. having only access to the header, a separate translation unit cannot instantiate a template at compile time. one must explicitly instantiate the templated type or function where it is defined, so that it can provide the symbols during linkage.
```cpp
// header.hpp
struct Test
{
    template <typename T>
    T get_default_value();
};
// template.cpp
template <typename T>
T get_default_value() {
    return T{};
}
/* explicit instantiation for int */
template int Test::get_default_value();
// main.cpp
#include "header.hpp"
int main()
{
    Test t;
    auto fdef = t.get_default_value<float>(); // undefined reference
    auto idef = t.get_default_value<int>();   // works
}
```
## 3.7. extern templates
for templated functions or types that require a considerable amount of compilation time, we can defer the instantiation to another translation unit by using the `extern` keyword. the instantation may be a normal use that would lead to implicit instantation, or an explicit instantiation:
```cpp
// header.hpp
template <typename T>
T large_function() {
    ...
}
// file1.cpp
bool some_test() {
    ...
    /* compiles large_function<double> */
    return large_function<double>() > 0;
}
// file2.cpp
extern template double large_function<double>();
```
# 4. Inheritance
## 4.1. friend functions and classes
a non-member function can access the private and protected members of a class if it is declared a friend of that class. that is done by including a declaration of this external function within the class, and preceding it with the keyword friend.
```cpp
class A
{
private:
    int a;
    friend void setA(A obj);
    friend class B;
};
void setA(A obj) {
  A.a=10;
};
class B
{
    void changeA(A obj) {
        obj.a = 14;
    }
};
```
## 4.2. access specifiers
| access                    | public | protected | private |
|:-------------------------:|:------:|:---------:|:-------:|
| members of the same class | y      | y         | y       |
| members of derived class  | y      | y         | n       |
| not members               | y      | n         | n       |

- public inheritance: all inherited members will keep their access specifiers in derived class
- protected inheritance : all inherited members will at most be proteted
- private inheritance : all inherited members will be private
## 4.3. what's inherited
* base' constructors and its destructor
* base' assignment operator members (operator=)
* base' friends
* base' private members
## 4.4. base constructor
base' default constructor is called with every child class object instantiation. a specific base constructor can also be called after colon when declaring child constructors.
```cpp
class Base
{
    public:
    Base() {
        std::cout << "default base constructor";
    }
    Base(int a) {
        std::cout << a << " base constructor";
    }
};
class Derived : public Base
{
public:
  Derived(int b) : Base(b) {}
}
```
if the base only has a non-derfault constructor, the child **must** have a constructor that calls the base constructor.
## 4.5. virtual functions
are made in the base class to be overridden in the derived class. they can still be called though. if a function is not defined as virtual and still overridden in the child class, a base class pointer that was initialized with the adress of a child object will always call the function in the base class (power of polymorphism goes to shit).

since C++11, it is recommended to add `override` after function declaration in the child class. This keyword helps the compiler detect if a virtual function has been overloaded, instead of overridden. for example if in the derived class, we forget the const keyword.
## 4.6. pure virtual and abstract classes
a pure virtual function is a virtual function whose definition is "=0". classes with at least one pure virtual function are called abstract. we cannot create objects of abstract classes but can create pointers for polymorphism. this also doesn't mean that they can't have useful callable functions.
```cpp
Base *base_ptr = new Child;
```
## 4.7. virtual destructors
```cpp
class Base
{
public:
    Base() { std::cout << "Base Constructor\n"; }
    ~Base() { std::cout << "Base Destructor\n"; }
};

class Derived : public Base
{
public:
    Derived() { memory = new int[500]; std::cout << "Derived Constructor\n"; }
    ~Derived() { delete memory; std::cout << "Derived Destructor\n"; }
private:
    int *memory;
}
int main()
{
    Base *base = new Base();
    delete base;
    std::cout << "---------------\n";
    Derived *derived = new Derived();
    delete derived;
    std::cout << "---------------\n";
    Base *poly = new Derived();
    delete poly;
}
output:
Base Constructor
Base Destructor
---------------
Base Constructor
Derived Constructor
Derived Destructor
Base Destructor
---------------
Base Constructor
Derived Constructor
Base Destructor
```
when calling the destructor for `poly`, we don't know that the object might have another destructor, since it's cast to `Base` pointer. to overcome this issue we have to declare the Base destructor as `virtual`. this does not override the base destructor. it just lets C++ know that there might be another destructor further down in the hierrachy.
## 4.8. virtual inheritance
if there is a diamond structure in the class hierarchy, the bottom class (`A`), gets two copies of `D`. one through `B` and another one through `C`.
```cpp
#include <iostream>

class D {
public:
    void foo() {
        std::cout << "foo" << std::endl;
    }
};

class C:  public D {
};

class B:  public D {
};

class A: public B, public C {
};

int main(int argc, const char * argv[]) {
    A a;
    a.foo();
}

```
to avoid this, B and C should use virtual inheritance.
```cpp
class C:  virtual public D {
};

class B:  virtual public D {
};
```
## 4.9. virtual tables and dynamic dispatch
a routine is created for each normal class method. the compiler remembers its address and dispatches it for all calls to this method. this is known as static dispatch or early binding since the correct address is known at compile time. this obviously fails with polymorphism.

for every class that contains or inherits virtual functions, the compiler constructs a virtual table. the vtable contains an entry for each virtual function accessible by the class and stores a pointer to its definition. entries in the vtable can point to either functions declared in the class itself (overridden), or virtual functions inherited from a base class.

vtables exist at the class level, meaning there exists a single vtable per class, and is shared by all instances. each class instance however has this vpointer added to its beginning. the pointer is 4 bytes or 8 bytes long depending on the cpu architecture.

virtual destrcutors become a clear necessity, since otherwise polymorphic objects would dispatch the base destructor statically.
## 4.10. type erasure
type erasure is a way of accomplishing the task usually done by polymorphism by hiding the type at runtime. the basic example of type erasure is `std::function` which can hold any type that is callable with a certain signature, be it a real function, a lambda, or a bound functor (e.g. bound to an pointer with `std::bind_front`. type erasure has the following benefits:<br>
- simpler and faster to compile interfaces
- can work with any feature type that adheres to the interface
- compilation firewall to prevent recompiling the entire library for adding a new type
Example:
```cpp
class animal_view {
public:
    template <typename Speakable>
    explicit animal_view(const Speakable &speakable)
        : object{&speakable},
            speak_impl{[](const void* obj) {
                return static_cast<const Speakable*>(obj)->speak();
            }} {}
    void speak() const { speak_impl(object); }
private:
    const void* object;
    void (*speak_impl)(const void*);
};

void do_animal_things(animal_view animal) { animal.speak() };

int main() {
    struct Cow {
        void speak() { std::cout << "moo!\n";
    };
    struct Sheep {
        void speak() { std::cout << "hello!\n";
    };
    do_animal_things(animal_view{Cow{}});
    do_animal_things(animal_view{Sheep{}});
}
```
we did polymorphism without polymorphism or dynamic allocations. we still do a function indriction, similar to dynamic dispatch. that means we gain no performance over virtual functions. all the code does get inlined and the entire call is replaced with a `jmp` to the function pointer. the downside of this approach is the risk of dangling pointers.
# 5. Type Conversion
- If a negative integer value is converted to an unsigned type, the resulting value corresponds to its 2's complement bitwise representation (i.e., -1 becomes the largest value representable by the type, -2 the second largest, ...).
- The conversions from/to bool consider false equivalent to zero (for numeric types) and to null pointer (for pointer types); true is equivalent to all other values and is converted to the equivalent of 1.
- If the conversion is from a floating-point type to an integer type, the value is truncated (the decimal part is removed). If the result lies outside the range of representable values by the type, the conversion causes undefined behavior.
- Otherwise, if the conversion is between numeric types of the same kind (integer-to-integer or floating-to-floating), the conversion is valid, but the value is implementation-specific (and may not be portable).
## 5.1. through classes
coverting A ojects to B and vice versa.
```cpp
class A {};
class B
{
public:
    // conversion from A (constructor):
    B (const A& x) {}
    // conversion from A (assignment):
    B& operator=(const A& x) {return *this;}
    // conversion to A (type-cast operator)
    operator A() {return A();}
};

int main ()
{
    A foo;
    B bar = foo;    // calls constructor
    bar = foo;      // calls assignment
    foo = bar;      // calls type-cast operator
    return 0;
}
```
the type-cast operator uses a particular syntax: it uses the operator keyword followed by the destination type and an empty set of parentheses. notice that the return type is the destination type and thus is not specified before the operator keyword.
## 5.2. explicit keyword
if we add the function `void fn (B arg) {}` to the previous example, it can be called with both `foo` and `bar`. to avoid such implicit casting, we should define the constructor as explicit : `explicit B (const A& x) {}`

additionally, constructors marked with explicit cannot be called with the assignment-like syntax; in the above example, bar could not have been constructed with: `B bar = foo`.

type-cast member functions (those described in the previous section) can also be specified as explicit. this prevents implicit conversions in the same way as explicit-specified constructors do for the destination type.
## 5.3. type casting
there are two main syntaxes for generic type casting:
- `c like     : int b = (int)a;`
- `functional : int b = int(a);`

unrestricted explicit type-casting allows to convert any pointer into any other pointer type, independently of the types they point to. the subsequent call to member result will produce either a run-time error or some other unexpected results.
### dynamic_cast
`dynamic_cast` can only be used with pointers and references to classes (or with void*). its purpose is to ensure that the result of the type conversion points to a valid complete object of the destination pointer type. this naturally includes pointer upcast (converting from pointer-to-derived to pointer-to-base), in the same way as allowed as an implicit conversion. but `dynamic_cast` can also downcast (convert from pointer-to-base to pointer-to-derived) polymorphic classes (those with virtual members) if -and only if- the pointed object is a valid complete object of the target type.
```cpp
class Base { virtual void dummy() {} };
class Derived: public Base { int a; };
int main ()
{
    Base *pba = new Derived;
    Base *pbb = new Base;
    Derived *pd;

    pd = dynamic_cast<Derived*>(pba); // works
    pd = dynamic_cast<Derived*>(pbb); // produces nullptr
}
```
even though both are pointers of type Base*, pba actually points to an object of type Derived, while pbb points to an object of type Base. therefore, when their respective type-casts are performed using `dynamic_cast`, pba is pointing to a full object of class Derived, whereas pbb is pointing to an object of class Base, which is an incomplete object of class Derived.

if `dynamic_cast` is used to convert to a reference type and the conversion is not possible, an exception of type bad_cast is thrown instead. `dynamic_cast` can also perform the other implicit casts allowed on pointers: casting null pointers between pointers types (even between unrelated classes), and casting any pointer of any type to a `void*`.

Google style suggests avoiding dynamic casts.
### static_cast
`static_cast` can perform conversions between pointers to related classes, not only upcasts (from pointer-to-derived to pointer-to-base), but also downcasts (from pointer-to-base to pointer-to-derived). no checks are performed during runtime to guarantee that the object being converted is in fact a full object of the destination type. therefore, it is up to the programmer to ensure that the conversion is safe. on the other side, it does not incur the overhead of the type-safety checks of `dynamic_cast`.
```cpp
class Base {};
class Derived: public Base {};
Base * a = new Base;
Derived * b = static_cast<Derived*>(a);
```
is a valid code but could lead to runtime errors if dereferenced.
### reinterpret_cast
reinterpret_cast converts any pointer type to any other pointer type, even of unrelated classes. the operation result is a simple binary copy of the value from one pointer to the other. All pointer conversions are allowed: neither the content pointed nor the pointer type itself is checked.

it can also cast pointers to or from integer types. the format in which this integer value represents a pointer is platform-specific. the only guarantee is that a pointer cast to an integer type large enough to fully contain it (such as intptr_t), is guaranteed to be able to be cast back to a valid pointer.
```cpp
class A { /* ... */ };
class B { /* ... */ };
A * a = new A;
B * b = reinterpret_cast<B*>(a);
```
this code compiles, although it does not make much sense, since now `b` points to an object of a totally unrelated and likely incompatible class. dereferencing `b` is unsafe.
### const_cast
this type of casting manipulates the constness of the object pointed by a pointer, either to be set or to be removed. for example, in order to pass a const pointer to a function that expects a non-const argument.
```cpp
void print (char * str) {
    std::cout << str << '\n';
}
int main ()
{
    const char *c = "sample text";
    print(const_cast<char *>(c));
    return 0;
}
```
the example above is guaranteed to work because function `print` does not write to the pointed object. note though, that removing the constness of a pointed object to actually write to it causes undefined behavior.
## 5.4. typeid
typeid allows to check the type of an expression: `typeid (expression)`

this operator returns a reference to a constant object of type type_info that is defined in the standard header `<typeinfo>`. a value returned by typeid can be compared with another value returned by typeid using operators == and != or can serve to obtain a null-terminated character sequence representing the data type or class name by using its name() member.
```cpp
#include <typeinfo>
int main () {
    int *a,b;
    a = 0; b = 0;
    if (typeid(a) != typeid(b)) {
        cout << "a and b are of different types:\n";
        cout << "a is: " << typeid(a).name() << '\n';
        cout << "b is: " << typeid(b).name() << '\n';
    }
    return 0;
}
```
when typeid is applied to classes, typeid uses the RTTI to keep track of the type of dynamic objects. when typeid is applied to an expression whose type is a polymorphic class, the result is the type of the most derived complete object.
```cpp
class Base { virtual void f(){} };
class Derived : public Base {};
int main () {
    Base* a = new Base;
    Base* b = new Derived;
    cout << "Base is: " << typeid(&Base).name() << '\n';        // Base
    cout << "Derived is: " << typeid(&Derived).name() << '\n';  // Derived
    cout << "a is: " << typeid(a).name() << '\n';               // Base*
    cout << "b is: " << typeid(b).name() << '\n';               // Base*
    cout << "*a is: " << typeid(*a).name() << '\n';             // Base
    cout << "*b is: " << typeid(*b).name() << '\n';             // Derived
}
```
the string returned by member name of `type_info` depends on the specific implementation of your compiler and library. it is not necessarily a simple string with its typical type name, like in the compiler used to produce this output. if the type `typeid` evaluates is a pointer preceded by the dereference operator (`*`), and this pointer has a null value, typeid throws a bad_typeid exception.
### mangled names
Depending on the object, typeid might return mangled names. GCC offers a demangle functionality. It can receive and populate a buffer but it can't resize it so there's basically no point to passing a pointer to length.
```cpp
#include <cxxabi.h>
char* abi::__cxa_demangle(const char* mangled_name, char* output_buffer, size_t* length, int* status);
```
Example (creates a memory leak):
```cpp
#include <typeinfo>
#include <cxxabi.h>
#include <vector>
#include <string>

struct ExStruct {
    std::vector<std::vector<std::string>> data;
};
int main () {
    int status;
    cout << "Base is: " << abi::__cxa_demangle(typeid(&ExStruct::data).name(), nullptr, 0, &status) << '\n';
    return 0;
}
```
## 5.5. boolean conversions
a value of any scalar type can be implicitly converted to `bool`. The values that compare equal to zero are converted to `0`, all other values are converted to `1`.
```cpp
bool b1 = 0.5;              // b1 == 1 (0.5 converted to int would be zero)
bool b2 = 2.0*_Imaginary_I; // b2 == 1 (but converted to int would be zero)
bool b3 = 0.0 + 3.0*I;      // b3 == 1 (but converted to int would be zero)
bool b4 = 0.0/0.0;          // b4 == 1 (NaN does not compare equal to zero)
```
## 5.6. integer conversions
a value of any integer type can be implicitly converted to any other integer type. except where covered by promotions and boolean conversions above, the rules are
* if the target type can represent the value, the value is unchanged
* otherwise, if the target type is unsigned, the value `2^b`, where b is the number of bits in the target type, is repeatedly subtracted or added to the source value until the result fits in the target type. in other words, unsigned integers implement modulo arithmetic.
    * otherwise, if the target type is signed, the behavior is implementation-defined (which may include raising a signal) 
```cpp
char x = 'a'; // int -> char, result unchanged
unsigned char n = -123456; // target is unsigned, result is 192 (that is, -123456+483*256)
signed char m = 123456;    // target is signed, result is implementation-defined
assert(sizeof(int) > -1);  // assert fails:
                           // operator > requests conversion of -1 to size_t,
                           // target is unsigned, result is SIZE_MAX
```
## 5.7. implicit arithmetic conversions
the arguments of the following arithmetic operators undergo implicit conversions for the purpose of obtaining the common real type, which is the type in which the calculation is performed:

* binary arithmetic `*`, `/`, `%`, `+`, `-`
* relational operators `<`, `>`, `<=`, `>=`, `==`, `!=`
* binary bitwise arithmetic `&`, `^`, `|`
* the conditional operator `?:`
### floating point conversions
if one of the operands has the base type long double, double or float, the other operand is also converted to this base type. e.g. `int > float` -> `float > float` and `complex > double` -> `double complex > double`.
### intergral conversions
integral types are never **implicitly** converted to lower integral types in a narrowing conversion. they may however be _promoted_ to to a common type.
```
if the types are the same:
    that type is the common type.
else:
    if the types have the same signedness (both signed or both unsigned):
        the operand whose type has the lesser conversion rank is implicitly converted to the other type.
    else:
        if the unsigned type has conversion rank greater than or equal to the rank of the signed type:
            the operand with the signed type is implicitly converted to the unsigned type.
        else:
            // the unsigned type has conversion rank less than the signed type
            if the signed type can represent all values of the unsigned type:
                the operand with the unsigned type is implicitly converted to the signed type.
            else:
                both operands undergo implicit conversion to the unsigned type counterpart of the signed operand's type.
```
### integer promotion rules
integer promotion is the implicit conversion of a value of any integer type with rank less or equal to rank of `int` or of a bit field of type `bool`, `int`, `signed int`, `unsigned int`, to the value of type `int` or `unsigned int`.

if `int` can represent the entire range of values of the original type (or the range of values of the original bit field), the value is converted to type `int`. otherwise the value is converted to `unsigned int`.

rank is a property of every integer type and is defined as follows:
1) the ranks of all signed integer types are different and increase with their precision: rank of `signed char` < rank of `short` < rank of `int` < rank of `long int` < rank of `long long int`
2) the ranks of all signed integer types equal the ranks of the corresponding unsigned integer types
3) the rank of any standard integer type is greater than the rank of any extended integer type of the same size (that is, rank of `__int64` < rank of `long long int`, but rank of `long long` < rank of `__int128` due to the rule (1))
4) rank of `char` equals rank of `signed char` and rank of `unsigned char`
5) the rank of `bool` is less than the rank of any other standard integer type
6) the rank of any enumerated type equals the rank of its compatible integer type
7) ranking is transitive: if rank of T1 < rank of T2 and rank of T2 < rank of T3 then rank of T1 < rank of T3
8) any aspects of relative ranking of extended integer types not covered above are implementation defined

Note: integer promotions are applied only
* as part of usual arithmetic conversions (see above)
* as part of default argument promotions
* to the operand of the unary arithmetic operators + and -
* to the operand of the unary bitwise operator ~
* to both operands of the shift operators << and >> 

# 6. Exception
thrown using `throw` keyword :
```cpp
int main ()
{
    try {
        throw 20;
    } catch (int e) {
        std::cout << "an exception occurred. Exception Nr. " << e << '\n';
    }
    return 0;
}
```
if an ellipsis `(...)` is used as the parameter of catch, that handler will catch any exception no matter what the type of the exception thrown.
## 6.1. specification
older code may contain dynamic exception specifications. they are now deprecated in C++, but still supported. a dynamic exception specification follows the declaration of a function, appending a throw specifier to it. for example:
```cpp
double myfunction (char param) throw (int);
```
this declares a function called myfunction, which takes one argument of type char and returns a value of type double. if this function throws an exception of some type other than int, the function calls ```std::unexpected``` instead of looking for a handler or calling ```std::terminate```. if this throw specifier is left empty with no type, this means that ```std::unexpected``` is called for any exception. functions with no throw specifier (regular functions) never call ```std::unexpected```, but follow the normal path of looking for their exception handler.
## 6.2. standard exceptions
the C++ Standard library provides a base class specifically designed to declare objects to be thrown as exceptions. it is called `std::exception` and is defined in the `<exception>` header. this class has a virtual member function called what that returns a null-terminated character sequence (of type `char*`) and that can be overridden in derived classes to contain some sort of description of the exception.
```cpp
#include <exception>
#include <iostream>
class myexception: public std::exception
{
    virtual const char* what() const throw() {
        return "My exception happened";
    }
} myex;

int main ()
{
    try {
        throw myex;
    } catch (std::exception& e) {
        std::cout << e.what() << '\n';
    }
    return 0;
}
```
standard exceptions are as follows :

| exception         | description                                            |
|-------------------|--------------------------------------------------------|
| bad_alloc         | thrown by new on allocation failure                    |
| bad_cast          | thrown by dynamic_cast when it fails in a dynamic cast |
| bad_exception     | thrown by certain dynamic exception specifiers         |
| bad_typeid        | thrown by typeid                                       |
| bad_function_call | thrown by empty function objects                       |
| bad_weak_ptr      | thrown by shared_ptr when passed a bad weak_ptr        |
| logic_error       | error related to the internal logic of the program     |
| runtime_error     | error detected during runtime                          |

`logic_error` and `runtime_error` are generic exceptions that can be inherited by custom exceptions to report errors.

*Google Style : don't use exceptions*

# 7. File IO
```cpp
#include <fstream>
using Mode = std::ios_base::openmode;
// input stream
std::ifstream f_in(std::string& filename, Mode mode);
// output stream
std::ofstream f_in(std::string& filename, Mode mode);
// IO stream
std::fstream f_in(std::string& filename, Mode mode);
```
## 7.1. modes

| mode                | effect                  |
|---------------------|-------------------------|
| `ios_base::app`     | append output           |
| `ios_base ::ate`    | seek to EOF when opened |
| `ios_base ::binary` | open in binary mode     |
| `ios_base ::in`     | open for reading        |
| `ios_base ::out`    | open for writing        |
| `ios_base ::trunc`  | overwrite existing file |

## 7.2. ascii reading

### using streams
```cpp
ifstream in ("file.txt", ios_base::in);
int a,b;
std::string str;
double d;
while (in >> a >> str >> b >> d) {
  ...
}
```
### getline
```cpp
ifstream in ("file.txt", ios_base::in);
std::string line;
int value;
while (getline(in, line)) {
    string::size_type loc = line.find("value", 0);
    if (loc != string::npos)
        value  = line.substr(line.find("=",0) + 1, string::npos);
}
```
## 7.3. ascii writing
```cpp
#include <iomanip>
#include <fstream>

ofstream outfile("file.txt");
if(!outfile.isopen())
    return EXIT_FAILURE;
outfile << "stuff" << std::endl;
```
## 7.4. binary reading
must know the structure beforehand, syntax :
```cpp
file.read(reinterpret_cast<char*> (&a), sizeof(a));
```
```cpp
#include <fstream>
#include <vector>

ifstream file("image.dat", ios_base::in | ios_base::binary);
if (!file) return EXIT_FAILURE;
int r = 0, c = 0;

file.read(reinterpret_cast<char*>(&r), sizeof(r));
file.read(reinterpret_cast<char*>(&c), sizeof(c));
std::cout<<"dimensions " << r << "x" << c;
vector<float> vec(r * c, 0);
file.read(reinterpret_cast<char*>(&vec.front()), vec.size()*sizeof(vec.front()));
```
## 7.5. binary writing
writing a sequence of byte, no precision loss for float types, fast. syntax:
```cpp
file.write(reinterpret_cast<char*> (&a), sizeof(a));
```
```cpp
#include <fstream>
#include <vector>

ofstream file("image.dat", ios_base::out | ios_base::binary);
if (!file) return EXIT_FAILURE;
int r = 2, c = 3;

vector<float> vec(r*c, 42);
file.write(reinterpret_cast<char*>(&r), sizeof(r));
file.write(reinterpret_cast<char*>(&c), sizeof(c));
file.write(reinterpret_cast<char*>(&vec.front()), vec.size()*sizeof(vec.front()));
```

# 8. Functors
function objects
```cpp
struct X
{
public:
    void  operator()(string str){
        std::cout << calling functor X with parameter << str << std::endl;
    }
};
int main()
{
    X foo;
    foo("Hi");
    return 0;
}
```
this is not to be mistaken with a type conversion function where the type comes after the `operator` keyword
```cpp
class X
{
    ...
    operator string() const { return "X"; }
};
```
## 8.1. pros
* normal functions are only called with the `()` operator but functors can also remember states.
* functors can have their own datatypes on top of their function signature
## 8.2. runtime templates
with templates we can customize how certain functions work, however their template has to be intialized at compile time. with functors we can do more
```cpp
struct AddValue
{
    int val_;
    AddValue(int j): val_(j) {}
    void operator()(int i){
        std::cout << i + val << std::endl;
    }
};
int main()
{
    std::vector<int> vec = {1, 5, 2, 3};
    int x;
    std::cin >> x;
    std::for_each(vec.begin(), vec.end(), AddValue(x));
}
```
## 8.3. built-in functors
std has implemented some functors:
```cpp
less greater greater_equal less_equal not_equal_to
logical_and logical_not logical_or
multiplies minus plus divide modulus negate
//example
int x = std::multiplies<int>()(3, 4);
if (not_equal_to<int>()(x, 10)) {
    std::cout << x << std::endl;
}
```
## 8.4. parameter binding
```cpp
std::set<int> myset = {2, 3, 4, 5};
std::vector<int> vec;

// code to multiply each element of the set
// by 10 and save in vector
std::transform(myset.begin(), myset.end(),  // source
               std::back_inserter(vec),     // destination
               std::bind(multiplies<int>(), std::placeholders::_1, 10));
```
transform takes a function that accepts one arguement, however `multiplies<int>()` accepts two. we use `std::bind` to replace the first parameter with a member of `myset` using `std::placeholders::_1`.<br> <br>

we can also use`std::bind` in the adding example. e.g. for adding with 2
```cpp
void AddVal(int i, int val){
    std::cout << i + val << std::endl;
}
std::for_each(vec.begin(), vec.end(), bind(AddVal, std::placeholders::_1, 2);
```
## 8.5. regular function to functor
```cpp
double Pow(double x, double y) {
    return pow(x,y);
}
int main()
{
    std::set<int> myset = {3, 1, 25, 7, 12};
    std::dequeue<int> d;
    auto func = std::function<double(double, double)>(Pow);
    std::transform(myset.begin(), myset.end(),
                    std::back_inserter(d),
                    std::bind(func, std::placeholders::_1, 2));
    return 0;
}
```
## 8.6. getting more complex
```cpp
bool ElgibileForCopy(int x) {
    return (x > 20) || (x < 5);
}
int main()
{
    std::set<int> myset = {3, 1, 25, 7, 12};
    std::dequeue<int> d;   // x in myset : x<20 || x<5
    auto func = std::function<double(double, double)>(Pow);
    std::transform(myset.begin(), myset.end(),
                   std::back_inserter(d),
                   std::bind(std::logical_or<bool>(
                             std::bind(greater<int>(), std::placeholders::_1, 20),
                             std::bind(less<int>(), std::placeholders::_1, 5)));
    // same as
    std::transform(myset.begin(), myset.end(),
                    std::back_inserter(d),
                    ElgibileForCopy); // probably won't work. it's a function not a functor
    return 0;
    // same as
    std::transform(myset.begin(), myset.end(),
                    std::back_inserter(d),
                    [] (int x) { return (x>20)||(x<5);});

}
```
## 8.7. functors and sorting
we can choose how our containers are sorted.
```cpp
std::set<int> myset = {3, 1, 25, 7, 12}; // myset : {1, 3, 7, 12, 25}
std::set<int, std::less<int>> myset = {3, 1, 25, 7, 12}; // myset : {1, 3, 7, 12, 25}

struct Lsb_less
{
    bool operator ()(int x, int y) {
        return (x % 10) < (y % 10);
    }
};
int main() {
    std::set<int, Lsb_less> myset = {3, 1, 25, 7, 12}; // myset : {1, 12, 3, 25, 7}
}
```
## 8.8. predicates
a predicate is a functor that
* returns a boolean
* does not modify data

predicates are used for comparisons or condition checks.
```cpp
struct NeedCopy
{
    bool operator()(int x) {
        return (x>20)||(x<5);
    }
};
std::transform(myset.begin(), myset.end(), std::back_inserter(d), NeedCopy());
```
## 8.9. template functors
to reuse defined functors to some extent, we can make use of templates
```cpp
template <typename T>
struct MatchFirstFunctor
{
    MatchFirstFunctor(T const& t) : m_t(t) {}
    template<typename U>
    bool operator()(U const& pair) {
        return pari.first == m_t;
    }
    T m_t;
}
int main() {
    std::vector<std::pair<int, bool>> v1 = {std::make_pair(1, true), std::make_pair(2, false)};
    std::vector<std::pair<bool, floa>> v2 = {std::make_pair(true, 1.0f), std::make_pair(false, 1.2f)};
    std::find(v1.begin(), v1.end(), std::make_pair(1, true));
    std::find_if(v1.begin(), v1.end(), MatchFirstFunctor<int>(2));
    std::find_if(v2.begin(), v2.end(), MatchFirstFunctor<bool>(false));
    return 0;
}
```
# 9. Iterators.
used to access the elements of collections. could be indexing a contiguous array or iterating a graph. types that have iteratos can be used in range-based for loops. Gemeral structure is as follows:
```cpp
template <typename Col>
class CollectionIterator {
public:
    using value_type = typename Col::value_type;
    using pointer_type = value_type*;
    using reference_type = value_type&;
    CollectionIterator(pointer_type ptr) : ptr_(ptr) {}
private:
    pointer_type ptr_;
};

template <typename T>
class Collection {
    using value_type = T;
    using iterator_type = CollectionIterator<Collection<T>>;
    ...
};
```
## 9.1. begin and end
the Collection class itself must implement the `begin()` and `end()` functions.
```cpp
iterator_type begin() {
    return iterator_type(data_);
}
iterator_type end() {
    return iterator_type(data_ + size_);
}
```
## 9.2. increment and decrement operators
for stepping over elements, the post/pre increment/decrement operators should be overloaded in the iterator class.
```cpp
CollectionIterator& operator++() {
    ptr_ += 1;
    return *this;
}
CollectionIterator operator++(int) {
    auto it = *this;
    ptr_ += 1;
    return it;
}
CollectionIterator& operator--() {
    ptr_ -= 1;
    return *this;
}
CollectionIterator operator--(int) {
    auto it = *this;
    ptr_ -= 1;
    return it;
}
```
## 9.3. index operator
the iterator class should also implement the index operator for random access:
```cpp
referemce_type operator[](size_t index) {
    return *(ptr_ + index);
}
```
## 9.4. arrow operator
the arrow operator should be overloaded in the iterator class to allow for member access. this operator has no inputs. it can techinically return anything, but should return something that either is a pointer or can become a pointer through chained `->` operators.  the `->` operator automatically dereferences its return value before calling its argument using the built-in pointer dereference, not `operator*`.
```cpp
pointer_type operator->() {
    return ptr_;
}
```
## 9.5. dereference operator
also should be overloaded by the iterator class:
```cpp
reference_type operator*() {
    return *ptr_;
}
```
## 9.6. comparison operators
required in the iterator class to terminate iteration.
```cpp
bool operator==(const CollectionIterator& other) const {
    return ptr_ == other.ptr_;
}
bool operator!=(const CollectionIterator& other) const {
    return ptr_ != other.ptr_;
}
```
# 10. Precompiled Headers
when we include a header file e.g. `vector`, we're potentially adding thousands of lines to be compiled each time we change anything in the project. this significantly increases compilation time. a precompiled header is already turned into a binary format that can easily be integrated into the chain.
## good practice
precompiled headers are mostly used for external code. for example we won't ever change STL headers or `windows.h`
## bad practice
a bad practice is to add all the dependencies to a PCH. this hides what is being used in each cpp file.<br>
putting frequently changing headers into a PCH will only increase compile time.
```c pp
//pch.h
#pragma once
#include <algorithm>
#include <vector>
#include <deque>
#include <stack>
#include <array>
#include <set>
#include <map>
#include <string>
#include <thread>
#include <memory>
#include <Windows.h>
```
## compile using gcc
we first compile the header, then the file that's using it.
```bash
g++ -std=c++11 pch.h
g++ -std=c++11 main.cpp
```
not supported by gcc. workarounds are bad.

# 11. IEEE 754 and caveats
a fixed-point binary format has a static position for the decimal point. the bits to its left represent the whole part while the right bits represent the fraction. fixed-point representation can vastly simplify the computation of opeartions but also limits range and precision.

a floating-point uses the scientific notation. they are represented by the IEEE 754 binary format on almost all modern architectures.

![image](https://user-images.githubusercontent.com/49957586/136662629-8d2330c1-58a5-4e16-9fe0-4f1c080eb220.png)

x = `-1^(sign bit) x 2^(exponent - 127) x 1.mantissa`
* **32-bit**: 1 sign bit + 8 exponent bits + 23 mantissa bits
* **64-bit**: 1 sign bit + 12 exponent bits + 52 mantissa bits

## 11.1. special floats
* divide by zero: `1 / 0`
* not a number (NaN): `0 / 0`, `0 x ∞`
* signed infinity: for overflow protection
* signed zero: underflow protection for very small numbers. preserves sign. `+0 == -0`

### basic example
```cpp
auto zeroPointOne = 0.1f;
auto zeroPointTwo = 0.2f;
auto zeroPointThree = 0.3f;
auto sum = zeroPointOne + zeroPointTwo;
std::cout << zeroPointOne << "\n";    // 0.100000000149...
std::cout << zeroPointTwo << "\n";    // 0.200000000298...
std::cout << zeroPointThree << "\n";  // 0.300000001192...
std::cout << sum << "\n";             // 0.300000001192...
```
how about using double instead?
```cpp
std::cout << zeroPointOne << "\n";    // 0.100000000000000005551115123126
std::cout << zeroPointTwo << "\n";    // 0.200000000000000011102230246252
std::cout << zeroPointThree << "\n";  // 0.299999999999999988897769753748
std::cout << sum << "\n";             // 0.300000000000000044408920985006
```
## 11.2. exponent encoding
1.0 = -1^0 x 2^(0) x 1.0
the exponent itself is an 8-bit unsigned integer. `2^(exponent - 127)` can therefor be between `2^-126` to `2^+127`. here `exponent - 127` must be zero, therefor the floating point representation will be: `[0][01111111][00000000000000000000000]`

## 11.3. epsilon
epsilon is the difference between `1.0` and smallest number that is larger than `1.0`. using the last example, it is `[0][01111111][00000000000000000000001] = 1.0000001192092896`. it is accessible using `std::numeric_limits<T>::epsilon` for floating-point type `T`.

## 11.4. significant digits
they measure precision. in order to compare two floating-point numbers solely based on the mantissa, they would have to have the same exponent. otherwise, the number of signficant digits changes based on the exponent. even though we will have a lot of decimal places, there is a measure of their accuracy and the rest is noise.

|          size          | sign | exponent | mantissa | total | exponent bias | bits precision | significant digits |
|:----------------------:|:----:|:--------:|:--------:|:-----:|:-------------:|:--------------:|:------------------:|
|          half          |   1  |     5    |    10    |   16  |       15      |       11       |         3-4        |
|         single         |   1  |     8    |    23    |   32  |      127      |       24       |         6-9        |
|         double         |   1  |    11    |    52    |   64  |      1023     |       53       |        15-17       |
| x86 extended precision |   1  |    15    |    64    |   80  |     16383     |       64       |        18-21       |
|          Quad          |   1  |    15    |    112   |  128  |     16383     |       113      |        33-36       |

## 11.5. denormalized numbers
for 32-bit floats, the minimum base 10 exponent is -36. how an we represent smaller numbers? 
since the exponent field is 127 left shift-encoded, we get the smallest value by setting all of its bits to zero (`2^(0 - 127) = 2^-127`). 

we can represent 1e-37 using _denormalized numbers_, i.e. numbers where the exponent bits are zero. this helps prevent underflow. example:
`[0][00000000][11011001110001111101110] = 0.09999999e-37`

## 11.6. precision
![image](https://user-images.githubusercontent.com/49957586/136664112-39a9b29b-2a67-40cb-829a-c6d6ea63374b.png)

as a result of their implementation, the representation is not uniform between numbers. we have the most precision between `0.0` and `0.1` (applies to negative values too). after this, the precision starts to diminish. `std::nextafter` allows us to get the very next possible number after the given one.

for 32-bit floats, the representable number distribution is as follows:
| from |  to | individual numbers |
|:----:|:---:|:------------------:|
|  0.0 | 0.1 |    1'036'831949    |
|  0.1 | 0.2 |      8'388'608     |
|  0.2 | 0.4 |      8'388'608     |
|  0.4 | 0.8 |      8'388'608     |
|  0.8 | 1.6 |      8'388'608     |
|  1.6 | 3.2 |      8'388'608     |
## 11.7. units in last place
pi represented as a floating point number has some errors:
```
pi  = 3.141589265
pif = 3.141589274
d   = 0.000000009
```
ulps is defined as the gap between two floating-point numbers nearest to x, even if x is one of them. for pi, we get a ulps of 9. IEEE 754 requires that elementary arithmetic operations are correctly rounded to within 0.5 ulps. transcendal functions are generally rounded to 0.5-1.0 ulps.
## 11.8. relative error
the difference between the real number and the approximated representation, devided by the real number.

`relative error = (pi - pif) / pi = 2.864789e-8`
## 11.9. rounding error
induced by approximating an infinite range of numbers into a finite number of bits. math is done exactly, then rounded:
* towards the nearest representable number
* towards zero
* towards positive infinity (round up)
* towards negative infinity (round down)

if there is a tie between rounding up & down, the number is rouneded to the one with the even mantissa. this avoids the bias of always rounding up or down.
## 11.10. extra bits
during calculations, 3 extra bits are considered per floating point number, although they are not saved in the float itself:
* gaurd bit
* rounding bit
* sticky bit

gaurd bit and rounding bit are extra precision. the sticky bit is an OR of anything that passes through it.
| [G][R][S] |             action            |
|:---------:|:-----------------------------:|
| [0][-][-] |    round down (do nothing)    |
| [1][0][0] | round up if mantissa lsb is 1 |
| [1][0][1] |            round up           |
| [1][1][0] |            round up           |
| [1][1][1] |            round up           |
## 11.11. algebraic assumption error
mathematical identites (associative, commutative, distributive) do not hold for floating point numbers.
* distributive: `x * y - x * z != x * (y - z)`
* associative: `x + (y + z) != (x + y) + z`
* cannot interchange division and multiplication: `x / 10.0 != x * 0.1`
## 11.12. floating point exceptions
| IEEE exception | result when traps disabled | argument to trap handler |
|:--------------:|:--------------------------:|:------------------------:|
|    overflow    |     +/- ∞ or +/- xmax      |     round(x2^-alpha)     |
|    underflow   | 0, 2^e_min or denormalized |      round(x2^alpha)     |
| divide by zero |           +/- ∞            |     invalid operation    |
|     invalid    |             NaN            |     invalid operation    |
|     inexact    |          round(x)          |         round(x)         |

where alpha is 192 for single precision, 1536 for double.
## 11.13. float tricks
there are some tricks for the IEEE specification. a famous one being quake's fast reciprocal square root approximation of numbers larger than 0.25:
```cpp
inline float fast_inv_sqrt(float x) {
  int tmp = ((0x3f800000 << 1) + 0x3f800000 - *(long*)&x) >> 1;
  auto y = *(float*)&tmp;
  return y * (1.47f - 0.47f * x * y * y);
}
```
