# Questions

* **What is `mutable` keyword used for ?** <br/> 
Members which are declared as mutable can be changed inside const functions/by const objects.

* **Derived class object assigned to Base class reference** <br/>
Unlike pointers, binding is done only once at initialization, which can't be changed again. Say we do this : 
```cpp
Base b1(10);
Derived d(20, 30);
Base &b = d;    // base class reference bound to derived class 
b = b1;         // contains parts of b1 & remaining parts of d; Devil class
```

* **What is object slicing? When does it occur?** <br/>
Assign ( = operator) a derived class object to a base class. Only base class members information is kept. 

* **What is the use of `override` keyword ?** <br/>
Gives compile time error, if we are overriding a non-virtual method of base class.

* **Deleting a nullptr/NULL. Is it safe/recommended ?** <br/>
`delete` operator anyways checks for NULL, so any null check before delete is redundant. Also, its okay not setting the pointer to NULL after delete. A program that is correct does not delete a pointer twice, and a program that does delete a pointer twice should crash. Also, no need if pointer is going out of scope. Set it to NULL for complicated lifecycles. 

* **Consider the below code. Does it work as expected & why/why not?** <br/>
```cpp
class A{
    std::string name;
    
    public:
    A(std::string n):name(n){}
}

A a("Shashank");
A a = "Shashank";
```
First one compiles - one implicit conversion from const char* to string. <br/>
Second doesn't - two implicit conversions. From const char* to string and then copy constructor overload called. *Two implicit conversions are not allowed.*

* **What is copy elision or copy initialization?** <br/>
Consider this code : `std::string s = "hi";` is equivalent to `std::string s = std::string("h1")` but most modern compilers will optimize it to avoid an extra copy from temp to `s`. Thus, code is interpreted as `std::string s("hi");`. But, two implicit conversions are not allowed. Thus, code in previous example doesn't compile

* **How to have a class which can be only be created by using `new` operator and compile error on creating instance directly ?**
By making Destructor private.

* **What is the size of an empty class ?**
An empty class has size of 1 byte to make sure that two different objects don't point to same memory address. However, if a class derives from empty base class compiler is free to make optimization that base class size is zero. 

* **What is the output of the following program/compiler error ?** 
```cpp
#include<iostream>
using namespace std;
 
class X 
{
public:
    int x;
};
 
int main()
{
    X a = {10};
    X b = a;
    cout << a.x << " " << b.x;
    return 0;
}
```
For solution, see [Question 6](https://www.geeksforgeeks.org/c-plus-plus-gq/constructors-gq/). Works for multiple types in list, but should not be virtual otherwise first 4 bytes are vtable.



# Key points

- Static members of a class must be initialized. Initialization is done outside the class. 
- Static member variables do not contribute in sizeof(class). They are present in data sections.
- Static methods can't access class's non-static members/methods. But if they get access to class ptr, then they can access, even private members.  
- private static member of a class can be used to launch a single instance at app launch. Especially useful for observer pattern, registering in constructor. 
- **Avoid bare pointers. Also do not mix match**
- A functions accepts arguments and wants to return a big object, eg matrix. Return options : 
    - By copy 
    - By bare pointer
    - By reference 
    - **By move constructor** (best) 
- Never throw while holding a resource not held by anyone else 
- C++14 : **requires** for templates to ensure we get what we expect & better error messages 
- `nullptr` is of type `std::nullptr_t` which implicitly converts to bool and any pointer. Use `nullptr` instead of `NULL`. Otherwise, 
  - avoid overloading on integers/numerical types and pointers. 
  - Passing `NULL` to templates deduces type as `int` and not pointer.

### Inheritance 
* **Important (Overriding vs virtual):** If a function is 'virtual' in the base class, the most-derived class's implementation of the function is called according to the actual type of the object referred to, regardless of the declared type of the pointer or reference. In non-virtual functions, the functions are called according to the type of reference or pointer. Virtual is what leads to runtime polymorphism
  * However in case object slicing occurs, then base class members are called. 
* *Pure Virtual functions can also have a body:* See - https://stackoverflow.com/questions/5481941/c-pure-virtual-function-have-body. Classes with pure virtual functions can't be instantiated but these can be called from derived class using fully-qualified names.
* There is nothing like Virtual Constructor. Making constructors virtual doesn't make sense as constructor is responsible for creating an object and it can’t be delegated to any other object by virtual keyword means.
* A destructor can be virtual. We may want to call appropriate destructor when a base class pointer points to a derived class object and we delete the object. If destructor is not virtual, then only the base class destructor may be called.
* **Important :** When a class has a virtual function, functions with same signature in all descendant classes automatically become virtual. We don't need to use virtual keyword in declaration explicitly. They are anyways virtual. Refer to this [Question 13](https://www.geeksforgeeks.org/c-plus-plus-gq/virtual-functions-gq/)
* Using virtual keyword while inheriting : `class Derived : virtual public Base1, virtual public Base2` solves diamond problem. Optimizes space. See [Question 3's](https://www.geeksforgeeks.org/c-plus-plus-gq/inheritance-gq/) explanation. 
* If a derived class writes its own method, then all functions of base class with same name become hidden, even if signaures of base class functions are different. See [Question 8](https://www.geeksforgeeks.org/c-plus-plus-gq/inheritance-gq/)
* 

### Final keyword - post C++11 
* `final` is not exactly a keyword. It holds meaning in 2 specific contexts otherwise can be used as a normal keyword. 
* `final` in front of `virtual` methods prevent them from being overridden further by child/derived classes. 
* `final` in front of class as in `class Test final {}` prevent the class/struct from being inherited. 


### Operator overloading 
* C++ compiler differentiates between overloaded postfix & prefix operators as postfix have a dummy parameter. 
* Insert operator '<<' should be overloaded as a global operator. 
* Overloading operator `new`, `delete` : See [Question 8](https://www.geeksforgeeks.org/c-plus-plus-gq/operator-overloading-gq/). Consider the following statement `Test *ptr = new Test;`. There are actually two things that happen in the above statement--memory allocation and object construction; the new keyword is responsible for both. One step in the process is to call operator new in order to allocate memory; the other step is to actually invoke the constructor. Operator new only allows us to change the memory allocation method, but does not do anything with the constructor calling method. Keyword new is responsible for calling the constructor, not operator new.

### Templates 
* We can pass non-type arguments to templates. Non-type parameters are mainly used for specifying max or min values or any other constant value for a particular instance of template. The important thing to note about non-type parameters is, they must be const. Compiler must know the value of non-type parameters at compile time. Eg., ```template <class T, int max>
int arrMin(T arr[], int n)```
* Attempting to modify value of non-type arguments of templates gives compiler error as these are const.  

# Type inference

Type inference happen for : (i) Templates, (ii) Auto. Before proceeding, a refresher on how to take arrays as references : 

  * `void fun(int (&arr)[5]); // It is mandatory to give size which is compile time constant.`
  * If we want to take array of any size as input, use templates : <br>
  `template<size_t N> void fun(int (&arr)[N]); // Here N is automatically deduced`
  * To take array of any size, any type as input reference, use : <br>
  `template<typename T, size_t N> void fun(T (&arr)[N]);`
  * Alternative : `template<typename T> void fun(T &arr);`. To compute length of array : `sizeof(arr)/sizeof(arr[0])`.

Also a quick referesher on **trailing return types**. In C++11, below two statements are equivalent : 
```cpp 
double compute_sq_root(int x);
auto compute_sq_root(int x) -> double; // hint to compiler about return type. 
```

## Template Type Deduction 

```cpp
template<Type T>
void fun(ParamType param)
```

#### 1. ParamType is reference or a pointer 

This is pretty straightforward, no surprises. Predict type for ParamType and accordingly fit T based on qualifiers in ParamType. Rvalues can't be passed here. Type of passed argument becomes ParamType

#### 2. ParamType is universal reference 

```cpp
tempate<typename Type> 
void fun(Type&& param)
```

*This is the only scenario where template Type can evaluate to a reference.* This happens in all cases where a lvalue is passed.

* **Question :** Consider the below code. Does it work as expected ? If not, why?
```cpp
template<typename Type>
Type compute_square(Type&& num) {
    Type square;
    square = num*num;
    return square
}

double x = 2.5;
double square = compute_square(x); cout << square << endl;
```

*Answer :* It gives compile error. `Type&&` translates to universal reference and hence `Type` is deduced to `double&`. References must be initialized in the line in which they are declared. Thus, compile error.

#### 3. ParamType is pass by value 

*Important thing to remember here is that if ParamType doesn't contain any qualifiers constness, volatileness, refrenceness are lost.*

* **Question :** What is the type deduced in the below case

```cpp
template<typename Type>
void fun(Type t)

const int * const x = 8;
fun(x)
```

*Answer :* `const int *` const of only pointer is removed and not where it points to. 

* **Question :** Write a function which takes an array of any size and type and returns its size ? 

*Answer :* 
```cpp
template <typename T, size_t N>
constexpr size_t array_size(T (&arr)[N]) noexcept {
    return N;       // also an example of how to pass arrays by reference
}
```

* **Question :** What is the deduced type in case of two functions below : 

```cpp
template <typename T>
void func1(T arr[])

template <typename T>
void func2<T& arr>

int arr[] = {1, 2, 3, 4};
func1(arr); func2(arr);
```

*Answer :* `int *` and `int [5]`. In `func2()` we can use `sizeof` in type T to determine size of array passed. That is how the function to compute size in case of arrays work

## Auto type deduction 

Auto type deduction is exactly same as template type deduction barring the case of `std::initializer_list<>`

```cpp
// 4 ways of initializing a variable

int x = 5;
int x(5);
int x = {5};        // uniform initialization
int x{5};

auto x = 5;             // int 
auto x(5);              // int 
auto x = {5};           // std::initializer_list<int>
auto x{5};              // std::initializer_list<int>
auto x = {1, 2, 3};     // std::initializer_list<int>
```

Passing initializer list in template type deduction will lead to compile error. C++14 allows us to used `auto` in return types and lamdas. Template type deduction is used when `auto` is used as return type in *C++14* or in lambda.

## Decltype for type deduction 

`decltype` deduces the exact type of the param without ignoring const-ness or volatile-ness. It can be used `decltype(variable_name)` or `decltype(auto)` (*latter only in C++14*)

```cpp
// Sample use 
int x;
vector<decltype(x)> v_x;

const int& cx = x;
auto y = cx;                // type(y) = int; Copy by value in template type deduction ignores reference and const-ness
decltype(cx) y = cx;        // type(y) = type(cx) = const int&
decltype(auto) y = cx;      // C++14 only 
```

**C++11 trailiing return types :** 

```cpp
auto prod(int x, int y) -> int;
int prod(int x, int y)              // equivalent

template<typename Type>
auto func(Type& x) -> decltype(x);
```

In C++11, trailing return types are often used with `decltype` as we now have the liberty of using the function parameters in return type. In C++14, we can directly use `auto`. In C++14, `decltype(auto)` is used to keep reference-ness. 

```cpp
template<typename Type>
auto func(Type& x){
    // do some logging 
    return x;
}

template<typename Type>
decltype(auto) func2(Type& x){
    // do some logging 
    return x;
}

int x = 5;
func(x) = 10;       // compile error as return type deduced is int (template type deduction)
func2(x) = 10;      // return type = type of x = int&
```

#### Printing typenames 

We can used `typeid(variable_or_type_name).name()` to get type name as string. But these results are compiler dependent and might not be accurate. Alternatively we can use `boost::typeindex::type_id_with_cvr<T>().pretty_name()` present in `boost\type_index.hpp>`. Examples below.

```cpp
cout << typeid(x).name();

template<typename T>
void printtype(T x) {
    cout << typeid(T).name();
}

boost::typeindex::type_id_with_cvr<decltype(x)>().pretty_name();
boost::typeindex::type_id_with_cvr<T>().pretty_name();
```

# Typecasting using operator overloading

Objects of class can impilicity cast to other types - basic or composite ie class using this. Syntax is : `operator typename(){}` Remember that such type casts do not have any return type or arguments.

```cpp
class Celsius{
    double c_temp;
    ...
    operator Farenheight(){
        return Farenheight(32 + c_temp*9/5);
    }
    operator double(){
        return c_temp;
    }
}

Celsius c(15);
Farenheight f = c; 
double temp = c;
```

<br>

# Initialization

### Default initialization 
Default initialization occurs in compiler generated default constructors and in functions. It leaves primitive types indeterminate and calls default constructor for composite types. Eg : 

```cpp 
void default_initialization() {
    int i;          // indeterminate
    std::mutex m;   // default constructor
    widget w;       // default constructor
    int a[9];       // indeterminate
}
``` 

### Aggregate initialization 
Aggregate initialization simply refers to initializing arrays using a list : `int a[3] = {1, 2, 3};` or `int a[3] = {}`. We can similarly initialize class/structs as well. 

### Value initialization
Value initialization refers to initializing values with default/0. One way is to do this manually : 
```cpp 
void manual_value_initialization() {
    int i=0;
    std::mutex m;
    Widget w = {};
    int a[9] = {};
}
```

Here value is initialized depending on the type. But we can't use this syntax in case of function templates as we are not sure whether type is primitive or composite. Also keep in mind C++ most vexing parse : 

```cpp
Widget wx(10);  // constructor call with arguments
Widget wx();    // function definition
Widget wx({10}); // constructor call
Widget wx({});  // default constructor call
```

Before C++11, we can perform value initialization of a class instance by using : `A a = A()` or `A* a = new A()`. This syntax invokes default constructor if user-defined default constructor is present. In case, it is not present, it does not invoke compiler generate default constructor. Rather, it performs value initialization.

### Uniform initialization 

C++11 introduces uniform initialization which provides a uniform syntax for value initialization : 

```cpp 
void cpp11_value_initialization() {
    int i {};
    std::mutex m {};
    Widget w {};
    int a[9] {};
}
```

This can be easily used with function templates as well. Uniform initialization prevents implicit narrowing conversion between implicit converting types such as `int`, `double`, etc. However,uniform/braced initialization has very strong affinity towards constructor overloaded with `std::initializer_list<>`. It calls other constructor only when types are not convertible.

**What is `std::initializer_list` ?**

`std::initializer_list<T>()` is nothing but a list of values of type `T`. It defines a constructor and member functions `begin()` and `end()` for iteration

## using keyword vs typedef

## static_assert 


## std::move

Move operator casts a value to its rvalue.

#### C++11 implementation

```cpp
template<typename T>            // in namespace std
std::remove_reference<T>::type&& move(T&& param) {
    using paramType = typename std::remove_reference<T>::type&&;
    return static_cast<paramType>(param);
}
```

#### C++14 implementation

```cpp
template<typename T>            // in namespace std
decltype(auto) move(T&& val){
    using paramtype = std::remove_reference_t<T>&&;
    return static_cast<paramtype>(val);
}
```

## std::forward 

Forward operator casts a value to its rvalue only if it receives a rvalue. It is often used to forward parameters to correct constructor depending on input. A classical use-case of `std::forward` looks as below : 

```cpp
template<typename T> 
void somefunc(T&& param) {
    // do some work
    somefunc2(std::forward<T>(param));
}
```

#### Implementation 

```cpp
// C++11
template<typename T>        // in namespace std
T&& forward(typename remove_refernce<T>::type& param){
    return static_cast<T&&>(param);
}

//C++14
template<typename T>
T&& forward(remove_reference_t<T>& param) {
    return static_cast<T&&>(param);
}
```
Todo : parameter pack 

## Perfect Forwarding constructor

Consider a customer class containing two strings, first name & last name. Constructor can be simply defined by using pass by value arguments and a const char * overload for copy initialization. Alternatively, we can use perfect forwarding constructor : 

#### C++11 

```cpp
class Cust {
    std::string first; 
    std::string last;

    public:
    template<typename S1, typename S2 = std::string, 
        typename = typename std::enable_if<!std::is_same<S1, Cust&>::value>::type >
    Cust(S1&& f, S2&& l=""):
        first(std::forward<S1>(f)), last(std::forward<S2>(l))
    {}
}
```

#### C++14 

```cpp
//...
    template<typename S1, typename S2 = std::string, 
        typename = std::enable_if_t<!std::is_same<S1, Cust&>::value> >
//...
```

#### C++17

```cpp
//...
    template<typename S1, typename S2 = std::string, 
        typename = std::enable_if_t<!std::is_same_v<S1, Cust&>>>
//...
```

Important points : 
* is_same is used to allow copy constructor to be called in case of non-const reference. 
* Important to use Cust& as this is the deduced type

However, all of this fails in case of inheritance. Suppose, a class VIP inherits Cust and we try to initialize Cust with object of child class. In this case deduced type is VIP& and copy constructor is not called 

#### C++17 - correct implementation 

```cpp
//...
    template<typename S1, typename S2 = std::string, 
        typename = std::enable_if_t<std::is_convertible_v<S1, std::string>>>
//...
```

#### C++20 

```cpp
class Cust {
    std::string first;
    std::string last; 

    public:
    template<typename S1, typename S2 = std::string, 
       requires std::is_convertible_v<S1, std::string>
    Cust(S1&& f, S2&& l=""):
        first(std::forward<S1>(f)), last(std::forward<S2>(l))
    {}
}
```

## Rule of 3 

If your class needs any of

* a copy constructor,
* an assignment operator,
* or a destructor,

defined explictly, then it is likely to need all three of them.

#### Implicit defintions

Let us consider a simple example of person class and its implicit definitions: 

```cpp 
class person
{
    std::string name;
    int age;
public:
    person(const std::string& name, int age) : name(name), age(age) {}
};

// Implicit definitions 

// 1. copy constructor
person(const person& that) : name(that.name), age(that.age) {}

// 2. copy assignment operator
person& operator=(const person& that) {
    name = that.name;
    age = that.age;
    return *this;
}

// 3. destructor
~person() {}
```
#### Explicit definitions 

The implicitly-defined special member functions are typically incorrect if the class is managing a resource whose handle is an object of non-class type (raw pointer, POSIX file descriptor, etc),whose destructor does nothing and copy constructor/assignment operator performs a "shallow copy" (copy the value of the handle, without duplicating the underlying resource). 

Sample code below. `new` can potentially throw exception. Exception safety should be guaranteed in constructors so as to prevent object from ending up in a bad state. 

```cpp

class person
{
    char* name;
    int age;
public:
    // the constructor acquires a resource:
    // in this case, dynamic memory obtained via new[]
    person(const char* the_name, int the_age)
    {
        name = new char[strlen(the_name) + 1];
        strcpy(name, the_name);
        age = the_age;
    }

    // the destructor must release this resource via delete[]
    ~person()
    {
        delete[] name;
    }
};

// 1. copy constructor
person(const person& that)
{
    name = new char[strlen(that.name) + 1];
    strcpy(name, that.name);
    age = that.age;
}

// 2. copy assignment operator
person& operator=(const person& that)
{
    if (this != &that)
    {
        delete[] name;
        // This is a dangerous point in the flow of execution!
        // We have temporarily invalidated the class invariants,
        // and the next statement might throw an exception,
        // leaving the object in an invalid state :(
        name = new char[strlen(that.name) + 1];
        strcpy(name, that.name);
        age = that.age;
    }
    return *this;
}

// 3. copy assignment operator with exception safety
person& operator=(const person& that)
{
    char* local_name = new char[strlen(that.name) + 1];
    // If the above statement throws,
    // the object is still in the same state as before.
    // None of the following statements will throw an exception :)
    strcpy(local_name, that.name);
    delete[] name;
    name = local_name;
    age = that.age;
    return *this;
}
```

For resources which can't be copied (such as mutexes, file handles), copy constructor, copy assignment operator overload should be deleted (or declared private - privates can still be accessed inside the class). 

#### Rule of 3 implementation from cppreference

```cpp
class rule_of_three
{
    char* cstring; // raw pointer used as a handle to a dynamically-allocated memory block
    rule_of_three(const char* s, std::size_t n) // to avoid counting twice
    : cstring(new char[n]) // allocate
    {
        std::memcpy(cstring, s, n); // populate
    }
 public:
    rule_of_three(const char* s = "")
    : cstring(s, std::strlen(s) + 1)
    {}
    ~rule_of_three()
    {
        delete[] cstring;  // deallocate
    }
    rule_of_three(const rule_of_three& other) // copy constructor
    : rule_of_three(other.cstring)
    {}
    rule_of_three& operator=(rule_of_three other) // copy assignment
    {
        std::size_t n{std::strlen(other.cstring) + 1};
        auto new_cstring = new char[n]; // allocate
        delete[] cstring;  // deallocate
        cstring = new_cstring;
        std::memcpy(cstring, other.cstring, n); // populate
        return *this;
    }
};
```

#### Copy-and-swap idiom

*Info:* `std::swap` is a library function which exchanges the values of two objects.

Any class that manages a resource (eg. smart pointer wrapper) often implements the Big Three. Copy assignment operator overload is the most tricky as it needs to avoid code duplication and provide exception safety. 

Conceptually, it works by using the copy-constructor's functionality to create a local copy of the data, then takes the copied data with a swap function, swapping the old data with the new data. The temporary copy then destructs, taking the old data with it. We are left with a copy of the new data. 

In order to use the copy-and-swap idiom, we need three things: a working copy-constructor, a working destructor (both are the basis of any wrapper, so should be complete anyway), and a swap function.

A swap function is a non-throwing function that swaps two objects of a class, member for member. We might be tempted to use std::swap instead of providing our own, but this would be impossible; std::swap uses the copy-constructor and copy-assignment operator within its implementation, and we'd ultimately be trying to define the assignment operator in terms of itself!

Follow this link and subsequently inner ones : https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom 

A sample swap function is given below which simply swaps all member objects : 

```cpp
class dumb_array
{
public:
    friend void swap(dumb_array& first, dumb_array& second) // nothrow
    {
        // enable ADL (not necessary in our case, but good practice)
        using std::swap;

        // by swapping the members of two objects,
        // the two objects are effectively swapped
        swap(first.mSize, second.mSize);
        swap(first.mArray, second.mArray);
    }
};
```

Once swap is implemented, we can have our overload look like below. It provides exception safety as if copy constructor fails, original object is not affected. Also, its a no throw. 

```cpp
dumb_array& operator=(const dumb_array& other)
{
    dumb_array temp(other);
    swap(*this, temp);

    return *this;
}
```

We can further optimize by making the temp copy in parameter itself : 
```cpp
dumb_array& operator=(dumb_array other) {
    swap(*this, other);
    return *this;
}
```

## Rule of 5

#### Definition from cppreference

```cpp
class rule_of_five
{
    char* cstring; // raw pointer used as a handle to a dynamically-allocated memory block
    rule_of_five(const char* s, std::size_t n) // to avoid counting twice
    : cstring(new char[n]) // allocate
    {
        std::memcpy(cstring, s, n); // populate
    }
 public:
    rule_of_five(const char* s = "")
    : rule_of_five(s, std::strlen(s) + 1)
    {}
    ~rule_of_five()
    {
        delete[] cstring;  // deallocate
    }
    rule_of_five(const rule_of_five& other) // copy constructor
    : rule_of_five(other.cstring)
    {}
    rule_of_five(rule_of_five&& other) noexcept // move constructor
    : cstring(std::exchange(other.cstring, nullptr))
    {}
    rule_of_five& operator=(const rule_of_five& other) // copy assignment
    {
         return *this = rule_of_five(other);
    }
    rule_of_five& operator=(rule_of_five&& other) noexcept // move assignment
    {
        std::swap(cstring, other.cstring);
        return *this;
    }
// alternatively, replace both assignment operators with 
//  rule_of_five& operator=(rule_of_five other) noexcept
//  {
//      std::swap(cstring, other.cstring);
//      return *this;
//  }
};
```

Based on copy and swap idiom above, assuming we have a friend swap function which does nothing more than a member by member swap of all members. Then we can rewrite our constructors/assignment operators overload as : 

```cpp
class rule_of_five {
    //...
    rule_of_five(const rule_of_five& other) : rule_of_five(other.cstring) {}    // copy ctor

    rule_of_five(rule_of_five&& other): rule_of_five() {                        // move ctor
        swap(*this, other);
    }

    rule_of_five& operator=(const rule_of_five& other) {                        // copy assignment operator
        rule_of_five temp(other);
        swap(*this, temp);
        return *this;
    }

    rule_of_five& operator=(rule_of_five&& other) {                             // move assignment operator
        swap(*this, other);
        return *this;
    }

    // Alternatively and better yet common definition for operator overloads
    rule_of_five& operator=(rule_of_five other) {
        swap(*this, other);
        return *this;
    }
}
```

* Perfect forwarding constructor, std::enable_if, std::enable_if_t, std::is_same, std::is_same_v, std::is_convertible, std::is_convertible_v

## Short String Optimizations (SSO)

In general, a typical string class allocates the storage for the string’s text dynamically from the heap, using new[]. New (which in turn calls malloc) are expensive. Thus, the std::string class, will reserve a small chunk of memory, a “small buffer” (typically 15 characters) embedded inside std::string objects, and when strings are small enough, they will be kept (deep-copied) in that buffer, without triggering dynamic memory allocations.

## C++11 vs C++14 

* C+11 can't have type deduction in return type. If return is `auto`, there must a trailing return type. C++14 can use `auto`. 
* C+11 can use `decltype(x)` or `decltype(type_name)` but not `decltype(auto)`, which is introduced in C++14.

## Return Value Optimization 

## Operator overloading 

## Questions for stackoverflow 

<to be added if any>

## Exception Handling 

* There is a special catch block called 'catch all' catch(…) that can be used to catch all types of exceptions. However, 'catch all' block should be the last catch block otherwise compiler error. 
* Implicit type conversion doesn't happen for primitive types in catch block 
* Derived class exceptions can be caught by Base class object. Hence, a derived class exception should be caught before a base class exception.
* Like Java, C++ library has a standard exception class which is base class - `std::exception` for all standard exceptions. All objects thrown by components of the standard library are derived from this class. Therefore, all standard exceptions can be caught by catching this type. 
* Specifying uncaught exceptions by a function : void fun(int *ptr, int x) throw (int *, int)
* When an exception is thrown, all objects created inside the enclosing try block are destructed before the control is transferred to catch block. Order of destruction is reverse of the order of construction. Any object which raised an unhandled exception in constructor is not destroyed. 


# References 

* [Value initialization with C++](https://akrzemi1.wordpress.com/2013/09/10/value-initialization-with-c/)
* [Copy and swap idiom in detail](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom)