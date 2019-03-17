# Questions

**Overridden vs virtual function call** 
Polymorphism means ability of different member function to be called depending on type of invoking object. But if virtual is not used, it will always call to member function for which object was defined and not what it points to.

**mutable keyword** 
Members which are declared as mutable can be changed inside const functions/const objects.

**Base class reference pointing to derived class**

Binding is done only once at initialization, which can't be changed again otherwise formation of devil class.

* **What is object slicing? When does it occur?** <br/>
Assign ( = operator) a derived class object to a base class. Only base class members information is kept. 

* **What is the use of `override` keyword ?** <br/>
Gives compile time error, if we are overriding a non-virtual method of base class.

* **Deleting a nullptr/NULL. Is it safe/recommended ?** <br/>
`delete` operator anyways checks for NULL, so any null check before delete is redundant. Also, its okay not setting the pointer to NULL after delete. A program that is correct does not delete a pointer twice, and a program that does delete a pointer twice should crash. Also, no need if pointer is going out of scope. Set it to NULL for complicated lifecycles. 

* Consider the below code. Does it work as expected & why?
```cpp
class A{
    std::string name;
    
    public:
    A(std::string n):name(n){}
}

A a("Shashank");
A a = "Shashank";
```
First one compiles - one implicit conversion from const char* to string
Second doesn't - two implicit conversions. From const char* to string and then copy constructor overload called. 

# Key points

- Static members of a class must be initialized. Initialization is done outside the class 
- private static member of a class can be used to launch a single instance at app launch. Especially useful for observer pattern, registering in constructor. 
- **Avoid bare pointers. Also do not mix match**
- A functions accepts arguments and wants to return a big object, eg matrix. Return options : 
    - By copy 
    - By bare pointer
    - By reference 
    - **By move constructor** (best) 
- Never throw while holding a resource not held by anyone else 
- C++14 : **requires** for templates to ensure we get what we expect & better error messages 
- **`nullptr` is of type `std::nullptr_t` which implicitly converts to bool and any pointer. Use `nullptr` instead of `NULL`. Otherwise, avoid overloading on integers/numerical types and pointers. Passing `NULL` to templates deduces type as `int` and not pointer.**


# Type inference

## Template Type Deduction 

```cpp
template<Type T>
void fun(ParamType param)
```

#### ParamType is reference or a pointer 

This is pretty straightforward, no surprises. Predict type for ParamType and accordingly fit T based on qualifiers in ParamType. Rvalues can't be passed here. Type of passed argument becomes ParamType

#### ParamType is universal reference 

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

#### ParamType is pass by value 

*Important thing to remember here is that if ParamType doesn't contain any qualifiers, const and volatile qualifiers are removed as arguments are passed by value and copy of bits take place*. 

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

<br>

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

<br>

## Decltype for type deduction 

`decltype` deduces the exact type of the param without ignoring const-ness or volatile-ness. It can be used `decltype(variable_name)` or `decltype(auto)` (*only in C++14*)

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

# Initializer_list and Uniform Initialization

*Not so important section. Hardly will be using initializer_list* 

**What is `std::initializer_list` ?**

`std::initializer_list<T>()` is nothing but a list of values of type `T`. It defines a constructor and member functions `begin()` and `end()` for iteration

Uniform initialization prevents implicit narrowing conversion between implicit converting types such as `int`, `double`, etc. It also solves C++ most vexing parse. However,uniform/braced initialization has very strong affinity towards constructor overloaded with `std::initializer_list<>`. It calls other constructor only when types are not convertible.

C++ most vexing parse : 
```cpp
Widget wx(10);  // constructor call with arguments
Widget wx();    // function definition
Widget wx({10}); // constructor call
Widget wx({});  // default constructor call
```

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

#### Rule of 3 implementation from cppreference (Implicit definition)

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
        std::swap(cstring, other.cstring);
        return *this;
    }
};
```

## Rule of 5

#### Implicit definition from cppreference

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

## Copy Initialization

```cpp
class Sample{
    int x;
    public:
    Sample(int x):x(x){}
}

Sample s = 5;

// This fails - Why ? 
class CustomString{
    std::string s;
public:
    CustomString(std::string s): s(s) {}
}

CustomString cstr = "Shashank"; 
```

* Perfect forwarding constructor, std::enable_if, std::enable_if_t, std::is_same, std::is_same_v, std::is_convertible, std::is_convertible_v

## Short String Optimizations (SSO)

In general, a typical string class allocates the storage for the string’s text dynamically from the heap, using new[]. New (which in turn calls malloc) are expensive. Thus, the std::string class, will reserve a small chunk of memory, a “small buffer” embedded inside std::string objects, and when strings are small enough, they will be kept (deep-copied) in that buffer, without triggering dynamic memory allocations.

## C++11 vs C++14 

* C+11 can't have type deduction in return type. C++14 uses `decltype(auto)`

## Return Value Optimization 

## Operator overloading 

## Questions for stackoverflow 

* Questions regarding rule of 3 & rule of 5 in c++
