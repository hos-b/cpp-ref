# C++ Reference ([Part 2](https://github.com/hos-b/cpp-ref/blob/master/modern.md))
1. [Types and Stuff](#1-types-and-stuff)<br>
  1.1. [decltype and auto](#11-decltype-and-auto)<br>
  1.2. [typedef and using](#12-typedef-and-using)<br>
  1.3. [unions](#13-unions)<br>
  1.4. [enums](#14-enums)<br>
  1.5. [volatile keyword](#15-volatile-keyword)<br>
  1.6. [static keyword](#16-static-keyword)<br>
  1.7. [mutable keyword](#17-mutable-keyword)<br>
  1.8. [extern keyword](#18-extern-keyword)<br>
  1.9. [typename keyword](#19-typename-keyword)<br>
  1.10. [type deduction with declval](#110-type-deduction-with-declval)<br>
  1.11. [storage classes](#111-storage-classes)<br>
2. [Pointers](#2-pointers)<br>
  2.1. [const keyword](#21-const-keyword)<br>
  2.2. [function pointers](#22-function-pointers)<br>
3. [Class Basics](#3-class-basics)<br>
  3.1. [other ways of instantiating](#31-other-ways-of-instantiating)<br>
  3.2. [member initialization](#32-member-initialization)<br>
  3.3. [static members](#33-static-members)<br>
  3.4. [const objects and functions](#34-const-objects-and-functions)<br>
  3.5. [template specialization](#35-template-specialization)
4. [Inheritance](#4-inheritance)<br>
  4.1. [friend functions and classes](#41-friend-functions-and-classes)<br>
  4.2. [access specifiers](#42-access-specifiers)<br>
  4.3. [what's inherited](#43-whats-inherited)<br>
  4.4. [base constructor](#44-base-constructor)<br>
  4.5. [virtual functions](#45-virtual-functions)<br>
  4.6. [pure virtual and abstract classes](#46-pure-virtual-and-abstract-classes)<br>
  4.7. [virtual destructors](#47-virtual-destructors)<br>
  4.8. [virtual inheritance](#48-virtual-inheritance)<br>
  4.9. [virtual tables and dynamic dispatch](#49-virtual-tables-and-dynamic-dispatch)
5. [Type Conversion](#5-type-conversion)<br>
  5.1. [through classes](#51-through-classes)<br>
  5.2. [explicit keyword](#52-explicit-keyword)<br>
  5.3. [type casting](#53-type-casting)<br>
  5.4. [typeid](#54-typeid)
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
11. [Precompiled Headers](#10-precompiled-headers)<br>

## 1. Types and Stuff
### 1.1. decltype and auto
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
auto also _never_ performs a conversion, making it sometimes better than writing down the type explicitly.

### 1.2. typedef and using
declaring new types :
```cpp
typdef char C;
using C = char;
```
#### type aliasing
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
### 1.3. unions
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
### 1.4. enums
```cpp
//numbered starting from 0
enum colors_t {BLACK, BLUE, GREEN, CYAN, RED, PURPLE, YELLOW, WHITE};
//numbered startin from 1
enum months_t { JANUARY=1, FEBRUARY, MARCH, ...};
```
#### enum class, enum struct
enum class or enum struct helps avoid dumb misakes by preventing unwanted int casts.
```cpp
enum class Colors {BLACK, BLUE, GREEN, CYAN, RED, PURPLE, YELLOW, WHITE};
Colors mycolor;
mycolor = Colors::BLUE;
if (mycolor == Colors::GREEN) mycolor = Colors::RED;
```

#### determine type size
```cpp
enum class EyeColor : char {blue, green, brown};
```
#### explicit values
usable for masks
```cpp
enum class EnumType {
  OPTION_1 = 10,
  OPTION_2 = 0xBE,
  ...
}
```
### 1.5. volatile keyword
volatile is a hint to the implementation to avoid aggressive optimization involving the object because the value of the object might be changed by means undetectable by an implementation. the compiler might sometimes optimize objects that affect program flow e.g. a loop. from the compiler's perspective the object could be changed by a static value but that might not be the case. to avoid such faults, the volatile keyword should be used.
### 1.6. static keyword
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
### 1.7. mutable keyword
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
### 1.8. extern keyword
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
### 1.9. typename keyword
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
### 1.10. type deduction with declval
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
### 1.11. storage classes
the storage class specifiers are a part of the decl-specifier-seq of a name's declaration syntax. together with the scope of the name, they control two independent properties of the name: its storage duration and its linkage. 

* auto: automatic storage duration
* register: automatic storage duration. also hints to the compiler to place the object in the processor's register. removed in C++17
* static: static or thread storage duration and internal linkage
* extern: static or thread storage duration and external linkage
* thread_local: thread storage duration
* mutable - does not affect storage duration or linkage

## 2. Pointers
### 2.1. const keyword
```cpp
int x;
      int *       p1 = &x;  // non-const pointer to non-const int
const int *       p2 = &x;  // non-const pointer to const int
      int * const p3 = &x;  // const pointer to non-const int
const int * const p4 = &x;  // const pointer to const int
const int * p2a = &x;  //      non-const pointer to const int
int const * p2b = &x;  // also non-const pointer to const int
```
#### google style
CamelCase starting with `k`.
### 2.2. function pointers
#### standalone functions
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
#### member functions
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

## 3. Class Basics
### 3.1. other ways of instantiating
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
### 3.2. member initialization
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
### 3.3. static members
#### variables
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
#### functions
they do not have an object of the class. they can access private members if they are passed to them as parameters. syntax:
```cpp
ClassName::MethodName(<params>)
```
### 3.4. const objects and functions
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
### 3.5. template specialization
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
notice that we precede the class name with template<> , including an empty parameter list. This is because all types are known and no template arguments are required for this specialization, but still, it is the specialization of a class template, and thus it requires to be noted as such.
When we declare specializations for a template class, we must also define all its members, even those identical to the generic template class, because there is no "inheritance" of members from the generic template to the specialization.

## 4. Inheritance
### 4.1. friend functions and classes
A non-member function can access the private and protected members of a class if it is declared a friend of that class. That is done by including a declaration of this external function within the class, and preceding it with the keyword friend.
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
### 4.2. access specifiers
| access                    | public | protected | private |
|:-------------------------:|:------:|:---------:|:-------:|
| members of the same class | y      | y         | y       |
| members of derived class  | y      | y         | n       |
| not members               | y      | n         | n       |

public inheritance: all inherited members will keep their access specifiers in derived class
protected inheritance : all inherited members will at most be proteted
private inheritance : all inherited members will be private
### 4.3. what's inherited
* base' constructors and its destructor
* base' assignment operator members (operator=)
* base' friends
* base' private members
### 4.4. base constructor
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
### 4.5. virtual functions
are made in the base class to be overridden in the derived class. they can still be called though. if a function is not defined as virtual and still overridden in the child class, a base class pointer that was initialized with the adress of a child object will always call the function in the base class (power of polymorphism goes to shit).

since C++11, it is recommended to add `override` after function declaration in the child class. This keyword helps the compiler detect if a virtual function has been overloaded, instead of overridden. for example if in the derived class, we forget the const keyword.
### 4.6. pure virtual and abstract classes
a pure virtual function is a virtual function whose definition is "=0". classes with at least one pure virtual function are called abstract. we cannot create objects of abstract classes but can create pointers for polymorphism. this also doesn't mean that they can't have useful callable functions.
```cpp
Base *base_ptr = new Child;
```
### 4.7. virtual destructors
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
### 4.8. virtual inheritance
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
### 4.9. virtual tables and dynamic dispatch
a routine is created for each normal class method. the compiler remembers its address and dispatches it for all calls to this method. this is known as static dispatch or early binding since the correct address is known at compile time. this obviously fails with polymorphism.

for every class that contains or inherits virtual functions, the compiler constructs a virtual table. the vtable contains an entry for each virtual function accessible by the class and stores a pointer to its definition. entries in the vtable can point to either functions declared in the class itself (overridden), or virtual functions inherited from a base class.

vtables exist at the class level, meaning there exists a single vtable per class, and is shared by all instances. each class instance however has this vpointer added to its beginning. the pointer is 4 bytes or 8 bytes long depending on the cpu architecture.

virtual destrcutors become a clear necessity, since otherwise polymorphic objects would dispatch the base destructor statically.

## 5. Type Conversion
- If a negative integer value is converted to an unsigned type, the resulting value corresponds to its 2's complement bitwise representation (i.e., -1 becomes the largest value representable by the type, -2 the second largest, ...).
- The conversions from/to bool consider false equivalent to zero (for numeric types) and to null pointer (for pointer types); true is equivalent to all other values and is converted to the equivalent of 1.
- If the conversion is from a floating-point type to an integer type, the value is truncated (the decimal part is removed). If the result lies outside the range of representable values by the type, the conversion causes undefined behavior.
- Otherwise, if the conversion is between numeric types of the same kind (integer-to-integer or floating-to-floating), the conversion is valid, but the value is implementation-specific (and may not be portable).
### 5.1. through classes
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
the type-cast operator uses a particular syntax: it uses the operator keyword followed by the destination type and an empty set of parentheses. Notice that the return type is the destination type and thus is not specified before the operator keyword.
### 5.2. explicit keyword
if we add the function `void fn (B arg) {}` to the previous example, it can be called with both `foo` and `bar`. to avoid such implicit casting, we should define the constructor as explicit : `explicit B (const A& x) {}`

additionally, constructors marked with explicit cannot be called with the assignment-like syntax; In the above example, bar could not have been constructed with: `B bar = foo`.

type-cast member functions (those described in the previous section) can also be specified as explicit. This prevents implicit conversions in the same way as explicit-specified constructors do for the destination type.
### 5.3. type casting
there are two main syntaxes for generic type casting:
- `c like     : int b = (int)a;`
- `functional : int b = int(a);`
unrestricted explicit type-casting allows to convert any pointer into any other pointer type, independently of the types they point to. The subsequent call to member result will produce either a run-time error or some other unexpected results.
#### dynamic_cast
dynamic_cast can only be used with pointers and references to classes (or with void*). Its purpose is to ensure that the result of the type conversion points to a valid complete object of the destination pointer type. This naturally includes pointer upcast (converting from pointer-to-derived to pointer-to-base), in the same way as allowed as an implicit conversion. But dynamic_cast can also downcast (convert from pointer-to-base to pointer-to-derived) polymorphic classes (those with virtual members) if -and only if- the pointed object is a valid complete object of the target type.
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
even though both are pointers of type Base*, pba actually points to an object of type Derived, while pbb points to an object of type Base. Therefore, when their respective type-casts are performed using dynamic_cast, pba is pointing to a full object of class Derived, whereas pbb is pointing to an object of class Base, which is an incomplete object of class Derived.

If dynamic_cast is used to convert to a reference type and the conversion is not possible, an exception of type bad_cast is thrown instead. dynamic_cast can also perform the other implicit casts allowed on pointers: casting null pointers between pointers types (even between unrelated classes), and casting any pointer of any type to a void* pointer.

Google style suggests avoiding dynamic casts.
#### static_cast
static_cast can perform conversions between pointers to related classes, not only upcasts (from pointer-to-derived to pointer-to-base), but also downcasts (from pointer-to-base to pointer-to-derived). No checks are performed during runtime to guarantee that the object being converted is in fact a full object of the destination type. Therefore, it is up to the programmer to ensure that the conversion is safe. On the other side, it does not incur the overhead of the type-safety checks of dynamic_cast.
```cpp
class Base {};
class Derived: public Base {};
Base * a = new Base;
Derived * b = static_cast<Derived*>(a);
```
is a valid code but could lead to runtime errors if dereferenced.
#### reinterpret_cast
reinterpret_cast converts any pointer type to any other pointer type, even of unrelated classes. The operation result is a simple binary copy of the value from one pointer to the other. All pointer conversions are allowed: neither the content pointed nor the pointer type itself is checked.

It can also cast pointers to or from integer types. The format in which this integer value represents a pointer is platform-specific. The only guarantee is that a pointer cast to an integer type large enough to fully contain it (such as intptr_t), is guaranteed to be able to be cast back to a valid pointer.
```cpp
class A { /* ... */ };
class B { /* ... */ };
A * a = new A;
B * b = reinterpret_cast<B*>(a);
```
This code compiles, although it does not make much sense, since now b points to an object of a totally unrelated and likely incompatible class. Dereferencing b is unsafe.
#### const_cast
This type of casting manipulates the constness of the object pointed by a pointer, either to be set or to be removed. For example, in order to pass a const pointer to a function that expects a non-const argument.
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
The example above is guaranteed to work because function print does not write to the pointed object. Note though, that removing the constness of a pointed object to actually write to it causes undefined behavior
### 5.4. typeid
typeid allows to check the type of an expression: `typeid (expression)`

This operator returns a reference to a constant object of type type_info that is defined in the standard header `<typeinfo>`. A value returned by typeid can be compared with another value returned by typeid using operators == and != or can serve to obtain a null-terminated character sequence representing the data type or class name by using its name() member.
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
when typeid is applied to classes, typeid uses the RTTI to keep track of the type of dynamic objects. When typeid is applied to an expression whose type is a polymorphic class, the result is the type of the most derived complete object.
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
Note: The string returned by member name of type_info depends on the specific implementation of your compiler and library. It is not necessarily a simple string with its typical type name, like in the compiler used to produce this output. If the type typeid evaluates is a pointer preceded by the dereference operator (*), and this pointer has a null value, typeid throws a bad_typeid exception.
#### mangled names
Depending on the object, typeid might return mangled names. GCC offers a demangle functionality. It can received a buffer but it can resize it so there's basically no point.
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
}
```

## 6. Exception
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
### 6.1. specification
older code may contain dynamic exception specifications. They are now deprecated in C++, but still supported. A dynamic exception specification follows the declaration of a function, appending a throw specifier to it. For example:
```cpp
double myfunction (char param) throw (int);
```
this declares a function called myfunction, which takes one argument of type char and returns a value of type double. If this function throws an exception of some type other than int, the function calls ```std::unexpected``` instead of looking for a handler or calling ```std::terminate```. if this throw specifier is left empty with no type, this means that ```std::unexpected``` is called for any exception. Functions with no throw specifier (regular functions) never call ```std::unexpected```, but follow the normal path of looking for their exception handler.
### 6.2. standard exceptions
the C++ Standard library provides a base class specifically designed to declare objects to be thrown as exceptions. It is called `std::exception` and is defined in the `<exception>` header. This class has a virtual member function called what that returns a null-terminated character sequence (of type char *) and that can be overridden in derived classes to contain some sort of description of the exception.
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

Also deriving from exception, header `<exception>` defines two generic exception types that can be inherited by custom exceptions to report errors:

| exception         | description                                            |
|-------------------|--------------------------------------------------------|
| logic_error       | error related to the internal logic of the program     |
| runtime_error     | error detected during runtime                          |

*Google Style : don't use exceptions*

## 7. File IO
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
### 7.1. modes

| mode                | effect                  |
|---------------------|-------------------------|
| `ios_base::app`     | append output           |
| `ios_base ::ate`    | seek to EOF when opened |
| `ios_base ::binary` | open in binary mode     |
| `ios_base ::in`     | open for reading        |
| `ios_base ::out`    | open for writing        |
| `ios_base ::trunc`  | overwrite existing file |

### 7.2. ascii reading

#### using streams
```cpp
ifstream in ("file.txt", ios_base::in);
int a,b;
std::string str;
double d;
while (in >> a >> str >> b >> d) {
  ...
}
```
#### getline
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
### 7.3. ascii writing
```cpp
#include <iomanip>
#include <fstream>

ofstream outfile("file.txt");
if(!outfile.isopen())
    return EXIT_FAILURE;
outfile << "stuff" << std::endl;
```
### 7.4. binary reading
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
### 7.5. binary writing
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

## 8. Functors
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
### 8.1. pros
* normal functions are only called with the `()` operator but functors can also remember states.
* functors can have their own datatypes on top of their function signature
### 8.2. runtime templates
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
### 8.3. built-in functors
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
### 8.4. parameter binding
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
### 8.5. regular function to functor
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
### 8.6. getting more complex
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
### 8.7. functors and sorting
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
### 8.8. predicates
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
### 8.9. template functors
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
## 9. Iterators.
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
## 10. Precompiled Headers
when we include a header file e.g. `vector`, we're potentially adding thousands of lines to be compiled each time we change anything in the project. this significantly increases compilation time. a precompiled header is already turned into a binary format that can easily be integrated into the chain.
### good practice
precompiled headers are mostly used for external code. for example we won't ever change STL headers or `windows.h`
### bad practice
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
### compile using gcc
we first compile the header, then the file that's using it.
```bash
g++ -std=c++11 pch.h
g++ -std=c++11 main.cpp
```
not supported by gcc. workarounds are bad.
