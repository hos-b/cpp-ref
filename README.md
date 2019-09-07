# C++ Reference
0. [Types and Stuff](#1-types-and-stuff)<br>
  1.1. [decltype and auto](#11-decltype-and-auto)<br>
  1.2. [typedef and using](#12-typedef-and-using)<br>
  1.3. [unions](#13-unions)<br>
  1.4. [enums](#14-enums)<br>
  1.5. [volatile keyword](#15-volatile-keyword)<br>
  1.6. [static keyword](#16-static-keyword)<br>
  1.7. [mutable keyword](#17-mutable-keyword)<br>
  1.8. [extern keyword](#18-extern-keyword)
2. [Pointers](#2-pointers)<br>
  2.1. [const keyword](#21-const-keyword)<br>
  2.2. [function pointers](#22-function-pointers)
3. [Dynamic Memory](#3-dynamic-memory)<br>
  3.1. [nothrow](#31-nothrow)
4. [Class Basics](#4-class-basics)<br>
  4.1. [new retarded ways of instantiating](#41-new-retarded-ways-of-instantiating)<br>
  4.2. [member initialization](#42-member-initialization)<br>
  4.3. [static members](#43-static-members)<br>
  4.4. [const objects and functions](#44-const-objects-and-functions)<br>
  4.5. [template specialization](#45-template-specialization)<br>
  4.6. [move constructor and move assignment](#46-move-constructor-and-move-assignment)<br>
  4.7. [but wait there's more](#47-but-wait-theres-more)
5. [Inheritance](#5-inheritance)<br>
  5.1. [friend functions and classes](#51-friend-functions-and-classes)<br>
  5.2. [access specifiers](#52-access-specifiers)<br>
  5.3. [what's inherited](#53-whats-inherited)<br>
  5.4. [base constructor](#54-base-constructor)<br>
  5.5. [virtual functions](#55-virtual-functions)<br>
  5.6. [pure virtual and abstract classes](#56-pure-virtual-and-abstract-classes)
6. [Type Conversion](#6-type-conversion)<br>
  6.1. [through classes](#61-through-classes)<br>
  6.2. [explicit keyword](#62-explicit-keyword)<br>
  6.3. [type casting](#63-type-casting)<br>
  6.4. [typeid](#64-typeid)
10. [Namespaces](#10-namespacs)
## 1. Types and Stuff
### 1.1. decltype and auto
```cpp
int a = 5;
decltype(a) b;  // type of a without initialization
auto c = a;     // type of a with initialization
```
### 1.2. typedef and using
declaring new types :
```cpp
typdef char C;
using C = char;
```
### 1.3. unions
```cpp
union mix_t {
  int l;
  struct {
    short hi;
    short lo;
    } s;
  char c[4];
} mix;
```
### 1.4. enums
```cpp
//numbered starting from 0
enum colors_t {black, blue, green, cyan, red, purple, yellow, white}; 
//numbered startin from 1 enum months_t { january=1, february, march, april,
                may, june, july, august, september, october, november, december} y2k;
```
real enum types that are neither implicitly convertible to int and that neither have enumerator values of type int, but of the enum type itself. enum class or enum struct :
```cpp
enum class Colors {black, blue, green, cyan, red, purple, yellow, white};
Colors mycolor;
mycolor = Colors::blue;
if (mycolor == Colors::green) mycolor = Colors::red;
```
to determine type size :
```cpp
enum class EyeColor : char {blue, green, brown}; 
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
int main(void) {
  int j;
  for (j = 0; j < 5; j++) {
    printf("Value of f(): %d\n", f());
  }
  return 0;
}
```
returns 0 1 2 3 4
### 1.7. mutable keyword
The mutable storage class specifier is used only on a class data member to make it modifiable even though the member is part of an object declared as const. You cannot use the mutable specifier with names declared as static or const, or reference members.
```cpp
class A
{
  public:
    A() : x(4), y(5) { };
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
used to specify the type and the existance of an object. once all of the source files have been compiled, the linker will resolve all of the references of to the one definition that it finds in one of the compiled source files. the definition of the object needs to have “external linkage”, which means that it needs to be declared outside of a function and without the static keyword.
```cpp
# header.h:
extern int gloabl_x;
void print_gloabl_x();
---source1.cpp---
#include "header.h"
int gloabl_X;
void main(){
  gloabl_x =5;
  print_global_x()
}
---source 2---
#include "header.h"
void print_gloabl_x(){
  std::cout<<global_x;
}
```
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
### 2.2. function pointers
```cpp
int addition (int a, int b)
{ return (a+b); }
int subtraction (int a, int b)
{ return (a-b); }
int call (int x, int y, int (*functocall)(int,int))
{
  int g;
  return (*functocall)(x,y);
}
int main ()
{
  int (*minus)(int,int) = subtraction;
  int m = operation (7, 5, addition);int a = 5;
  int n = operation (20, m, minus);
}
```
## 3. Dynamic Memory
### 3.1. nothrow
```cpp
int *ptr = new (nothrow) int[1000000000];
if (ptr==nullptr) cout<<"producing null pointer instead runtime exception";
```
## 4. Class Basics
### 4.1. new retarded ways of instantiating
```cpp
class Circle {
  double radius;
  public:
  Circle(double r) { radius = r; }
};
Circle foo (10.0);   // functional form
Circle bar = 20.0;   // assignment init.
Circle baz {30.0};   // uniform init.
Circle qux = {40.0}; // POD-like
```
which also means 
```cpp
Rectangle rectb;   // default constructor called
Rectangle rectc(); // function declaration (default constructor NOT called)
Rectangle rectd{}; // default constructor called
```
### 4.2. member initialization
For members of fundamental types, it makes no difference how the constructor is defined, because they are not initialized by default, but for member objects (those whose type is a class), if they are not initialized using member initializer list, they are default-constructed:
```cpp
class A {
  double _r;
  public:
  A(double r) : _r(r) { }
};
class B {
  A base;
  public:
  Cylinder(double r) : base {r} {} //let's be retarded
};
```
or just make a pointer to A like a decent person.
### 4.3. static members
```cpp
class Dummy{
  static int counter;
  Dummy(){counter++;}
}
Dummy::counter = 0;
```
to avoid them being declared several times, they cannot be initialized directly in the class, but need to be initialized somewhere outside it.
### 4.4. const objects and functions
```cpp
const Class object;
```
* access to its data members from outside the class is restricted to read-only, as if all its data members were const for those accessing them from outside the class. member functions should be const if they are to be called outside the class.
* member functions specified to be const cannot modify non-static data members nor call other non-const member functions. In essence, const members shall not modify the state of an object.
* const objects are limited to access only member functions marked as const, but non-const objects are not restricted and thus can access both const and non-const member functions alike.
Most functions taking classes as parameters actually take them by const reference, and thus, these functions can only access their const members, i.e. ensuring that the passed object does not changed. overloading constness is also a thing :
```cpp
class MyClass {
  int x;
  public:
  MyClass(int val) : x(val) {}
  const int& get() const {return x;}
  int& get() {return x;}
};
```
### 4.5. template specialization
if the class uses templates but for a special data type, it should be declared a bit differently, we declare it twice. once with template classes, and once with an empty template and the specific data type:
```cpp
// class template:
template <class T>
class mycontainer {
  T element;
  public:
  mycontainer (T arg) {element=arg;}
  T increase () {return ++element;}
};
// class template specialization:
template <>
class mycontainer <char> {
  char element;
  public:
  mycontainer (char arg) {element=arg;}
  char uppercase ()
  {
    if ((element>='a')&&(element<='z'))
    element+='A'-'a';
    return element;
  }
};
```
notice that we precede the class name with template<> , including an empty parameter list. This is because all types are known and no template arguments are required for this specialization, but still, it is the specialization of a class template, and thus it requires to be noted as such. 
When we declare specializations for a template class, we must also define all its members, even those identical to the generic template class, because there is no "inheritance" of members from the generic template to the specialization.
### 4.6. move constructor and move assignment
for copy contstructor,assignment we had to define the following:
```cpp
MyClass::MyClass (const MyClass&);    // copy-constructor
MyClass& operator= (const MyClass&);  // copy-constructor
```
*similar to copying*, moving also uses the value of an object to set the value to another object. But, unlike copying, the content is actually transferred from one object (the source) to the other (the destination): the source loses that content, which is taken over by the destination. This moving only happens when the source of the value is an unnamed object. The move constructor and move assignment are members that take a parameter of type rvalue reference to the class itself:
```cpp
MyClass::MyClass (MyClass&&);   // move-constructor
MyClass& operator= (MyClass&&); // move-assignment
```
An rvalue reference is specified by following the type with two ampersands (&&). As a parameter, an rvalue reference matches arguments of temporaries of this type. the concept of moving is most useful for objects that manage the storage they use, such as objects that allocate storage with new and delete. In such objects, copying and moving are really different operations:
- Copying from A to B means that new memory is allocated to B and then the entire content of A is copied to this new memory allocated for B.
- Moving from A to B means that the memory already allocated to A is transferred to B without allocating any new storage. It involves simply copying the pointer.
in general :
```cpp
MyClass fn();            // function returning a MyClass object
MyClass foo;             // default constructor
MyClass bar = foo;       // copy constructor
MyClass baz = fn();      // move constructor
foo = bar;               // copy assignment
baz = MyClass();         // move assignment
```
Notes:

Compilers already optimize many cases that formally require a move-construction call in what is known as Return Value Optimization. Most notably, when the value returned by a function is used to initialize an object. In these cases, the move constructor may actually never get called.

Note that even though rvalue references can be used for the type of any function parameter, it is seldom useful for uses other than the move constructor. Rvalue references are tricky, and unnecessary uses may be the source of errors quite difficult to track.
### 4.7. but wait, there's more
Member function	      implicitly defined:	                                                  default definition:
Default constructor	  if no other constructors	                                            does nothing
Destructor	          if no destructor	                                                    does nothing
Copy constructor	    if no move constructor and no move assignment	                        copies all members
Copy assignment	      if no move constructor and no move assignment	                        copies all members
Move constructor	    if no destructor, no copy constructor and no copy nor move assignment	moves all members
Move assignment	      if no destructor, no copy constructor and no copy nor move assignment	moves all members
they're not always defined due to backward compatibility. if we want them to be (or not to be), we have to explicitly ask for an implicit declaration (or removal thereof) :
```cpp
function_declaration = default;
function_declaration = delete;
```
for example after this decalaration, the rectangle class can no longer use the copy constructor :
```cpp
class Rect{
  Rect (const Rect& other) = delete;
  ...
};
Rect b;
Rect a(b);  // error
```
in general, and for future compatibility, classes that explicitly define one copy/move constructor or one copy/move assignment but not both, are encouraged to specify either delete or default on the other special member functions they don't explicitly define.
## 5. Inheritance

### 5.1. friend functions and classes
A non-member function can access the private and protected members of a class if it is declared a friend of that class. That is done by including a declaration of this external function within the class, and preceding it with the keyword friend.
```cpp
class A:{
  private:
  int a;
  friend void setA(A obj);
  friend class B;
}
void setA(A obj){
  A.a=10;
}
class B{
  void changeA(A ojb){
    obj.a=14;
  }
}
```
### 5.2. access specifiers
Access	                    public	protected	private
members of the same class	  yes	    yes	      yes
members of derived class	  yes	    yes	      no
not members	                yes	    no        no

public inheritance: all inherited members will keep their access specifiers in derived class
protected inheritance : all inherited members will at most be proteted
private inheritance : all inherited members will be private
### 5.3. what's inherited
base' constructors and its destructor
base' assignment operator members (operator=)
base' friends
base' private members

### 5.4. base constructor
base' default constructor is called with every child class object instantiation. a specific base constructor can also be called after colon when declaring child constructors.
```cpp
class Base{
  public:
  Base() {std::cout<<"default base constructor";
  Base(int a) {std::cout<<a<<" base constructor";
};
class Derived : public Base{
  public:
  Derived(int b) : Base(b)
}
```
### 5.5. virtual functions
are made in the base class to be overwritten in the derived class. they can still be called though. if a function is not defined as virtual and still overwritten in the child class, a base class pointer that was initialized with the adress of a child object will always call the function in the base class (power of polymorphism goes to shit).

### 5.6. pure virtual and abstract classes
a pure virtual function is a virtual function whose definition is "=0". classes with at least one pure virtual function are called abstract. we cannot create objects of abstract classes but can create pointers for polymorphism. this also doesn't mean that they can't have useful callable functions.
```cpp
Base *base_ptr = new Child;
```
## 6. Type Conversion
- If a negative integer value is converted to an unsigned type, the resulting value corresponds to its 2's complement bitwise representation (i.e., -1 becomes the largest value representable by the type, -2 the second largest, ...).
- The conversions from/to bool consider false equivalent to zero (for numeric types) and to null pointer (for pointer types); true is equivalent to all other values and is converted to the equivalent of 1.
- If the conversion is from a floating-point type to an integer type, the value is truncated (the decimal part is removed). If the result lies outside the range of representable values by the type, the conversion causes undefined behavior.
- Otherwise, if the conversion is between numeric types of the same kind (integer-to-integer or floating-to-floating), the conversion is valid, but the value is implementation-specific (and may not be portable).

### 6.1. through classes
coverting A ojects to B and vice versa.
```cpp
class A {};
class B {
public:
  // conversion from A (constructor):
  B (const A& x) {}
  // conversion from A (assignment):
  B& operator= (const A& x) {return *this;}
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
### 6.2. explicit keyword
if we add the function `void fn (B arg) {}` to the previous example, it can be called with both `foo` and `bar`. to avoid such implicit casting, we should define the constructor as explicit : `explicit B (const A& x) {}`

additionally, constructors marked with explicit cannot be called with the assignment-like syntax; In the above example, bar could not have been constructed with: `B bar = foo`.

type-cast member functions (those described in the previous section) can also be specified as explicit. This prevents implicit conversions in the same way as explicit-specified constructors do for the destination type.

### 6.3. type casting
there are two main syntaxes for generic type casting:
- `c like     : int b = (int)a;`
- `functional : int b = int(a);`
unrestricted explicit type-casting allows to convert any pointer into any other pointer type, independently of the types they point to. The subsequent call to member result will produce either a run-time error or some other unexpected results.
#### dynamic_cast
dynamic_cast can only be used with pointers and references to classes (or with void*). Its purpose is to ensure that the result of the type conversion points to a valid complete object of the destination pointer type. This naturally includes pointer upcast (converting from pointer-to-derived to pointer-to-base), in the same way as allowed as an implicit conversion. But dynamic_cast can also downcast (convert from pointer-to-base to pointer-to-derived) polymorphic classes (those with virtual members) if -and only if- the pointed object is a valid complete object of the target type.
```cpp
class Base { virtual void dummy() {} };
class Derived: public Base { int a; };
void main () {
    Base *pba = new Derived;
    Base *pbb = new Base;
    Derived *pd;

    pd = dynamic_cast<Derived*>(pba); // works
    pd = dynamic_cast<Derived*>(pbb); // produces nullptr
}
```
even though both are pointers of type Base*, pba actually points to an object of type Derived, while pbb points to an object of type Base. Therefore, when their respective type-casts are performed using dynamic_cast, pba is pointing to a full object of class Derived, whereas pbb is pointing to an object of class Base, which is an incomplete object of class Derived.

If dynamic_cast is used to convert to a reference type and the conversion is not possible, an exception of type bad_cast is thrown instead. dynamic_cast can also perform the other implicit casts allowed on pointers: casting null pointers between pointers types (even between unrelated classes), and casting any pointer of any type to a void* pointer. 
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
void print (char * str)
{
  cout << str << '\n';
}

int main () {
  const char * c = "sample text";
  print ( const_cast<char *> (c) );
  return 0;
}
```
The example above is guaranteed to work because function print does not write to the pointed object. Note though, that removing the constness of a pointed object to actually write to it causes undefined behavior
### 6.4. typeid
typeid allows to check the type of an expression: `typeid (expression)`

This operator returns a reference to a constant object of type type_info that is defined in the standard header `<typeinfo>`. A value returned by typeid can be compared with another value returned by typeid using operators == and != or can serve to obtain a null-terminated character sequence representing the data type or class name by using its name() member.
```cpp
#include <typeinfo>
int main () {
  int * a,b;
  a=0; b=0;
  if (typeid(a) != typeid(b))
  {
    cout << "a and b are of different types:\n";
    cout << "a is: " << typeid(a).name() << '\n';
    cout << "b is: " << typeid(b).name() << '\n';
  }
  return 0;
}
```
when typeid is applied to classes, typeid uses the RTTI to keep track of the type of dynamic objects. When typeid is applied to an expression whose type is a polymorphic class, the result is the type of the most derived complete object.
```
class Base { virtual void f(){} };
class Derived : public Base {};
void main () {
    Base* a = new Base;
    Base* b = new Derived;
    cout << "a is: " << typeid(a).name() << '\n';     // Base*
    cout << "b is: " << typeid(b).name() << '\n';     // Base*
    cout << "*a is: " << typeid(*a).name() << '\n';   // Base
    cout << "*b is: " << typeid(*b).name() << '\n';   // Derived
}
```
Note: The string returned by member name of type_info depends on the specific implementation of your compiler and library. It is not necessarily a simple string with its typical type name, like in the compiler used to produce this output. If the type typeid evaluates is a pointer preceded by the dereference operator (*), and this pointer has a null value, typeid throws a bad_typeid exception.
## 10. Namespaces
### 10.1. hidden namespaces
```cpp
namespace multiply
{
    namespace // anonymous namespace
    {
        int first = 4;
        int second = 5;

        int calc(int a, int b)
        {
            return a * b;
        }
    }

    int getFirst()
    {
        return first;
    }

    int getSecond()
    {
        return second;
    }

    int getProduct()    
    {
        return calc(first, second);
    }
}
```
calc is hidden from the user.
