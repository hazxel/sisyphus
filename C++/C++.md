# Raw Expressions

- 逗号表达式，逗号表达式的优先级最低，`(a, b)`这个表达式的值就是`b`



# Pointer

##### Address related operator

- Address-of operator `&`
- Dereferencing operator `*`

##### Pointers and arrays

Arrays work very much like pointers to their first elements, and, actually, an array can always be implicitly converted to the pointer of the proper type.

Brackets `[]` dereference the variable just as `*`. The following are equivalent:

```c++
int a[10];
a[5] = 0;
*(a+5) = 0;
```

##### NULL vs nullptr

`nullptr` is introduced to resolve the ambiguity of `NULL`, because it's hard to tell if NULL is pointer or number. `nullptr` is a compile time constant, with type `nullptr_t`, and can be implicitly converted to any pointer type.

```c++
#define NULL ((void*)0) // definition in C
#define NULL 0			// definition in C++
typedef decltype(nullptr) nullptr_t;
```

##### void pointer

Can point to any kinds of variable. Some compiler forbid arithmatic (+/-) operation to void pointers. C allows a `void*` pointer to be assigned to any pointer type without a cast, while C++ does not.

##### memory operations `memcpy` & `memset` (Standard C Library)

Never use `memcpy` to copy classes. (e.g. copy unique_ptr)

Never use `memset` to reset classes. (e.g. vptr set to zero, resulting nullptr error) (`bzero` is deprecated)



# lvalue vs rvalue

|         lvalue         |         rvalue         |
| :--------------------------------------: | :--------------------------------------: |
|  points to a specific memory location  | in memory or register, can't get address |
| live a longer life since they exist as variables |    temporary and short lived     |
| can be left or right operand of assigment | can only be left right operand of assigment |
|    ++i (return reference of i)    | i++ (i has increased, return a saved temp value) |

- Values are either *glvalue* (generalized left value) or *rvalue* (right value). 
  - *glvalue*  inlcudes *lvalue* and *xvalue*
  - *rvalue* includes *prvalue* and *xvalue*
- Values are either lvalue ((non-expiring) left value), *prvalue* (pure right value) or *xvalue* (expiring value)
  - *lvalue*: 能够用&取地址的表达式，以及字符串字面值
  - *prvalue*: C++11之前的右值指的是C++11后的纯右值，包括字符串以外的所有字面值，不具名临时对象等
  - *xvalue*: 。C++11中的将亡值是随着右值引用的引入而引入的。所谓的将亡值表达式，就是下列表达式：
    - 返回右值引用的函数的调用表达式
    - 转换为右值引用的转换函数的调用表达式(`move`, `forward`)





# Reference

#### Reference vs Pointer

- Same: they are both related to the address:
 - A pointer points to a memory address
 - A reference is an alias of a memory address 
- Difference:
 - Initialization:
  - Reference must be initialized, and **cannot be modified to refer to other objects**.
  - Pointer can be initialized anytime, and can be modified anytime
 - Reference cannot be null, pointer can be null
 - No const reference (but reference to a const object is ok)
 - reference type must match with the object it is referring to (const reference is an exception)
 - increament operator (++) and sizeof() are different
  - Size of reference is the size of the object; size of the pointer is the size of the pointer itself
  - increament of reference increases the object, increament of the pointer makes the pointer point to the next address
 - reference has type check (safer)

#### rvalue refrence (&&)

本来引用不可以使用另一个引用初始化，但因为 c++11 引入了右值引用，现在可以

C++11 标准中规定，通常情况下右值引用形式的参数只能接收右值，不能接收左值。但对于函数模板中使用右值引用语法定义的参数来说，它不再遵守这一规定，既可以接收右值，也可以接收左值（此时的右值引用又被称为“万能引用” universal reference）。

> It might be confusing that lvalue reference and rvalue refrence themselves can be either lvalue or rvalue.

#### move & forward (C11)

> introduced in C11

- `std::move` (move semantic, 移动语义) is used to indicate that an object may be "moved from", i.e. allowing the efficient transfer of resources from t to another object. 

- `std::forward<T>` (perfect forwarding, 完美转发) When t is a forwarding reference (a function argument that is declared as an rvalue reference to a cv-unqualified function template parameter), this overload forwards the argument to another function with the value category it had when passed to the calling function. 

 C++11 标准中规定，通常情况下右值引用形式的参数只能接收右值，不能接收左值。但对于函数模板中使用右值引用语法定义的参数来说，它不再遵守这一规定，既可以接收右值，也可以接收左值（此时的右值引用又被称为“万能引用”）。

 ```c++
 template<class T>
 void wrapper(T&& arg)
 {
   // arg is always lvalue
   foo(std::forward<T>(arg)); // Forward as lvalue or as rvalue, depending on T
 }
 ```

They both do type conversion, implemented using `static_cast`. *Efficient Modern C++* suggests that use `std::move` for rvalue reference and `std:forward` for universal (only in template functions, `&&` reference can take either rvalue or lvalue) reference.

#### reference collapsing

For the following code:

```c++
typedef [SOME_TYPE] T;
typedef [SOME_TYPE] TR;
TR var;
```

the acutal type of var is:

| T  | TR | Type of var |
| :--: | :--: | :---------: |
| A& | T& |   A&   |
| A& | T&& |   A&   |
| A&& | T& |   A&   |
| A&& | T&& |   A&&   |

 #### reference qualifier 引用限定 C11

 ```c++
template <typename T>
class optional {
 // version of value for non-const lvalues
 constexpr T& value() &；
 // version of value for const lvalues
 constexpr T const& value() const&；
 // version of value for non-const rvalues... are you bored yet?
 constexpr T&& value() &&；
 // you sure are by this point
 constexpr T const&& value() const&&；
};
 ```





# const (Keyword)

- const variable: read-only

 > const object cannot call non-const member functions. 
 >
 > member of const object cannot be modified.

- const pointer:

 ```c++
 char greeting[] = "Hello";
 char* p1 = greeting;       // 指针变量，指向字符数组变量
 const char* p2 = greeting;    // 指针变量，指向字符数组常量（const后面是char，说明char只读）
 char* const p3 = greeting;    // 指针常量，指向字符数组变量（const后面是p3，说明p3只读）
 const char* const p4 = greeting; // 指针常量，指向字符数组常量
 ```

- const reference: refrence itself cannot be modified after initialization, so there are only references of const objects, no "const references".

 ```c++
 char c = 'a';
 const int &cr = c; // compiles
 int &ncr = c;    // won't compile
 const int &n = 5;  // compiles
 int &n = 5     // won't compile
 ```

- const member function:

 - such member function cannot modify member variables. (指针指向的对象是例外)
 - const instance can only call const member functions.
 - const member function cannot be static (static functions are independent on instances)

- const return type: useful when returning a reference of class' internal member



# static (keyword)

- static variables
 - normal variables are stored on stack
 - static variables are stored on static segment (scope remains the same), and invisible to other files
- static funcitons: only visible in current file, can be used to avoid conflicts
- static member variables: all instances share one copy, can be accessed without instances 
- static member functions: can be accessed without instances, but cannot use non-static members

# Global variable (C++ only, different from C)

```c++
int n = 3; // fileA.cpp, definition of non-const global variable
extern n; // fileB.cpp, global variable with external linkage
extern const int i = 4; // fileA.cpp, extern const definition
extern const int i; // fileB.cpp, declaration only 
```

- global variable is defined outside the function
- `extern` specifies that the variable or function is defined in another translation unit. 
- for **non-const** global variables, the `extern` keyword must be applied in all files **except** the one where the variable is defined.
- for **const** global variables, the `extern` keyword should be applied in all files
- The idea of const global variables is to replace the old C style `#define` for constants.



# Keywords

### noexcept(keyword)

The `noexcept` operator performs a compile-time check that returns true if an expression is declared to not throw any exceptions.

### explicit (keyword)

The `explicit` keyword is used to mark **constructors** or **conversion functions** to not implicitly convert types in C++. It is optional for **constructors that take exactly one argument** and **conversion functions** since those are the only functions that can be used in typecasting.

### typedef (keyword)

The `typedef` keyword is used for aliasing existing data types, user-defined data types, and pointers to a more meaningful name. Typedefs allow you to give descriptive names to standard data types, which can also help you self-document your code.

```c++
typedef std::vector<int> vInt;
```

### new/delete vs malloc/free

- `malloc/free`: 

 - STL function of C/C++
 - only allocate memory, no initialization
 - if success, return `void*` pointers, require casting
 - if fail, return null pointer

- `new/delete`: 

 - **operator** of C++ (can even be overridden to private)

 - Allocate memory and also call **default** constructors/ free memory and call destructors

  > require default constructor, but can use "allocator" to call other constructors if default one doesn't exist 

 - if success, return pointers of the corresponding type; if fail, throw exception

 > placement new: 允许向 new 传递额外的地址参数，从而在预先指定的内存区域创建对象。
 >
 > ```c++
 > new (place_address) type
 > new (place_address) type (initializers)
 > new (place_address) type [size]
 > new (place_address) type [size] { braced initializer list }
 > ```

 > **new[] / delete[]:**
 >
 > For creating arrays of instances. 若是自定义的数据类型，用new []申请的空间，必须要用delete []来释放，因为delete []会逐一调用对象数组的析构函数，然后释放空间，如果用delete，则只会调用第一个对象的析构函数，但空间还是会被释放

### sizeof (compile time operator)

- size of an object of an **empty class** is 1, in order to "ensure that the addresses of two different objects will be different." And the size can be 1 because alignment doesn't matter here, as there is nothing to actually look at.
- Existence of **virtual function(s)** will add 4 bytes of a virtual table pointer in the class. In this case, if the base class of the class already has virtual function(s) either directly or through its base class, then this additional virtual function won't add anything to the size of the class. Virtual table pointer will be common across the class hierarchy.
- Using of **virtual inheritance** will add 4 bytes of a virtual base table pointer in the class. A class will only maintain one virtual base table pointer.
- For classes, only non-static data members will be counted.
- alignment: alignment with the "widest" **basic** type (maybe inside a compound type), and compound types are treated as a whole ({double, char}, char -> 24)
- ordering matters here because of byte padding (char, short, int -> 8 ; char, int, short -> 12)
- char: 1, short: 2, int: 4, double: 8 (depends on GCC version, platform, etc.)

### constexpr (keyword)

> The keyword constexpr was introduced in C++11 and improved in C++14. It means constant expression.

Unlike `const`, `constexpr` can also be applied to functions and class constructors. `constexpr` indicates that the value, or return value, is constant and, where possible, is computed at compile time. When a value is computed at compile time instead of run time, it helps your program run faster and use less memory. (low latency!)

when a constexpr function is called with only compile-time arguments, the result of the function will be computed at compile-time. If, however, any argument is notknown at compile-time, the computation will be executed at runtime, like a regular function.



# cast

- static_cast: 这个操作符相当于C语言中的强制类型转换的替代品。多用于**非多态类型**的转换，比如说将int转化为double。但是不可以将两个无关的类型互相转化。（在编译时期进行转换）

 `static_cast` is used for cases where you basically want to reverse an implicit conversion, with a few restrictions and additions. static_cast performs no runtime checks. This should be used if **you know** that you refer to an object of a specific type, and thus a check would be unnecessary.

- dynamic_cast: 可以安全的将父类转化为子类，子类转化为父类都是安全的。所以你可以用于安全的将基类转化为继承类，而且可以知道是否成功，如果强制转换的是指针类型，失败会返回NULL指针，如果强制转化的是引用类型，失败会抛出异常。dynamic_cast 转换符只能用于含有虚函数的类, because it uses virtual funciton table to trace super class.

 `dynamic_cast` is useful when **you don't know** what the dynamic type of the object is. It returns a null pointer if the object referred to doesn't contain the type casted to as a base class (when you cast to a reference, a bad_cast exception is thrown in that case).

- const_cast: const_cast这个操作符可以去掉变量const属性或者volatile属性的转换符，这样就可以更改const变量了

- reinterpret_cast: 重新解释（无理）转换。即要求编译器将两种无关联的类型作转换。



# Runtime Type Information（RTTI）

In C++, RTTI can be used to do safe typecasts using the dynamic_cast<> operator, and to manipulate type information at runtime using the typeid operator and std::type_info class.

RTTI是Runtime Type Identification的缩写，意思是运行时类型识别。C++引入这个机制是为了让程序在运行时能根据基类的指针或引用来获得该指针或引用所指的对象的实际类型。但是现在RTTI的类型识别已经不限于此了，它还能通过typeid操作符识别出所有的基本类型（int，指针等）的变量对应的类型。

C++通过以下的两个操作提供RTTI：

1. typeid运算符，该运算符返回其表达式或类型名的实际类型
2. dynamic_cast运算符，该运算符将基类的指针或引用安全地转换为派生类类型的指针或引用



# Namespace

Namespaces provide a method for preventing name conflicts in large projects.

##### declaration for all the identifiers in a name space

Do not do this in header files! If others include your header file, it means they are also using namespace std, which may lead to conflicts! 

```c++
using namespace std;
```

##### declaration for single identifier

Only import one member of the namespace, recommended.

```c++
using std::cout;
```

> **Not namespace:**
>
> `using BaseClassName::BaseClassName;`
>
> For every constructor of BaseClass, compiler generates an identical constructor for derived class.

#### operator ::

- `::name` global namespace

- `class-name::member-name` scope is some class

 > can be used to call super class' virtual functions!

- `namespace-name::member-name` scope is some namespace

- Default: nearest scope



# RAII (Resource acquisition is initialization)

RAII 机制是一种对资源申请、释放这种成对的操作的封装，通过这种方式实现在局部作用域内申请资源然后便可以自动销毁资源



# New Features (C11, C17, C23)

### decltype (specifier)

Inspects the declared type of an entity or the type and value category of an expression.

```c++
struct A { double x; };
const A* a;
 
decltype(a->x) y;    // type of y is double (declared type)
decltype((a->x)) z = y; // type of z is const double& (lvalue expression)
 
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) { // return type depends on template parameters
  return t + u;            // return type can be deduced since C++14
}
```

> **Tailing Return Type Syntax:**
>
> The trailing return type feature removes a C++ limitation where the return type of a function template cannot be generalized if the return type depends on the types of the function arguments. For example, `a`and `b`are arguments of a function template `multiply(const A &a, const B &b)`, where `a`and `b`are of arbitrary types. Without the trailing return type feature, you cannot declare a return type for the `multiply`function template to generalize all the cases for `a*b`. With this feature, you can specify the return type after the function arguments. This resolves the scoping problem when the return type of a function template depends on the types of the function arguments.

### auto

The `auto` keyword directs the compiler to deduce the type of a variable from its initialization expression.

- Using auto drops references `&`, `const` qualifiers, and `volatile` qualifiers.
- use `auto` to simplify for iteration
- use `auto` to declare and initialize a variable to a lambda expression.
- Use `auto` together with `decltype` to write template libraries.

### decltype(auto)

more accurate type deduction than auto.



### lambda expression

A convenient way of defining an anonymous function, full syntax:

`[capture list] (params list) mutable exception-> return type { function body }`

but usually:

- `[capture list] {function body}`

- `[capture list] (param list) {function body}`

- `[capture list] (param list) -> return type {function body}`

 ##### Capture list

 Lambda expression can use the "outer" variables in it's scope, but must be included in the capture list `[]`

 pure lambda (non-capturing) expressions are free of side effects, and therefore cannot cause, e.g., race conditions 

捕获列表会形成一个闭包，实现原理呢就是靠语法糖生成一个匿名的结构体，捕获的都会作为这个匿名结构体中的变量

 ```c++
 void abssort(float* x, unsigned n) {
   std::sort(x, x + n,
     // Lambda expression begins
     [](float a, float b) {
       return (std::abs(a) < std::abs(b));
     } // end of lambda expression
   );
 }
 ```

  

 ```c++
 shared_ptr<vector<Data>> data;
 auto fun1 = [&]() {
 	//do somthing with data 1 million times
 };
 fun1();
 // 使用捕获列表时，会多一次内存寻址：需要先把被捕获的对象的地址存在栈上，再把该栈上存地址的单元的地址作为入参传入，同样的，在函数内访问该捕获对象时也需要取地址两次
  
 auto fun2 = [](shared_ptr<vector<Data>> &d) {
 	//do somthing with data 1 million times
 };
 // 通过参数传入引用时，和普通函数无区别
 fun2(data);
 ```

> 在使用 gcc 编译时，捕获列表为空，似乎也还是会安排一个字节的空间到栈上占位置:
>
> The current object (*this) can be implicitly captured if either capture default is present. If implicitly captured, it is always captured by reference, even if the capture default is `=`. The implicit capture of *this when the capture default is `=` is deprecated.(since C++20)

##### generic lambda (C++14)

generic lambda has `auto` in its parameter list, it's equivalent to:

```c++
auto lambda = [](auto x, auto y) {return x + y;};

struct unnamed_lambda
{
  template<typename T, typename U>
    auto operator()(T x, U y) const {return x + y;}
};
auto lambda = unnamed_lambda();
```

？？？泛型闭包：

```c++
auto f3 = [](auto a) {
  return [=]() mutable { return a = a + a; };
};
auto twice1 = f3(1);
cout << twice1() << endl; // 2
cout << twice1() << endl; // 4
auto twice2 = f3(string{"a"});
cout << twice2() << endl; // aa
cout << twice2() << endl; // aaaa
```



### Range-based for loop (since C++11)

```c++
for (auto declaration : expression) {
	// loop-statement
}
// is equivalant to:
auto && __range = range-expression;
auto __begin = begin-expr;
auto __end = end-expr;
for ( ; __begin != __end; ++__begin)
{
  range-declaration = *__begin;
  //loop-statement
}
```

- Choose `auto x` when you want to work with copies.
- Choose `auto &x` when you want to work with original items and may modify them.
- Choose `auto const &x` when you want to work with original items and will not modify them.

### Initializer list

统一初始化是使用大括号进行初始化的方式，其实是利用一个事实：编译器看到{t1, t2, …, tn}便会做出一个initializer_list，它关联到一个array<T, n>。调用构造函数的时候，该array内的元素会被编译器分解逐一传给函数。但若函数的参数就是initializer_list，则不会逐一分解，而是直接调用该参数的函数。

所有的标准容器的构造函数都有以initializer_list为参数的构造函数。initizlizer_list的最广泛的使用就是不定长度同类型参数的情况。
