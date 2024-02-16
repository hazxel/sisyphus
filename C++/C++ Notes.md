
### Pointer

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

Never use `memset` to reset classes. (e.g. vptr set to zero, resulting nullptr error)  (`bzero` is deprecated)

### lvalue vs rvalue

|                  lvalue                  |                  rvalue                  |
| :--------------------------------------: | :--------------------------------------: |
|   points to a specific memory location   | in memory or register, can't get address |
| live a longer life since they exist as variables |        temporary and short lived         |
| can be left or right operand of assigment | can only be left right operand of assigment |
|       ++i (return reference of i)        | i++ (i has increased, return a saved temp value) |

?Values are either *glvalue* (generalized left value) or *rvalue* (right value). 

?Values are either lvalue ((non-expiring) left value), *prvalue* (pure right value) or *xvalue* (expiring value)

### Reference

##### Reference vs Pointer

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


##### rvalue refrence (&&)

C++11 标准中规定，通常情况下右值引用形式的参数只能接收右值，不能接收左值。但对于函数模板中使用右值引用语法定义的参数来说，它不再遵守这一规定，既可以接收右值，也可以接收左值（此时的右值引用又被称为“万能引用” universal reference）。

> It might be confusing that lvalue reference and rvalue refrence themselves can be either lvalue or rvalue.

##### move & forward (C11)

- `std::move` (move semantic, 移动语义) is used to indicate that an object may be "moved from", i.e. allowing the efficient transfer of resources from t to another object. 

- `std::forward<T>` (perfect forwarding, 完美转发) When t is a forwarding reference (a function argument that is declared as an rvalue reference to a cv-unqualified function template parameter), this overload forwards the argument to another function with the value category it had when passed to the calling function. 

  ```c++
  template<class T>
  void wrapper(T&& arg) {
      // arg is always lvalue, 有名字一定是左值
      foo(std::forward<T>(arg)); // Forward as lvalue or as rvalue, depending on T
  }
  ```

They both do type conversion, implemented using `static_cast`. *Efficient Modern C++* suggests that use `std::move` for rvalue reference and `std:forward` for universal (only in template functions, `&&` reference can take either rvalue or lvalue) reference.

##### reference collapsing

For the following code:

```c++
typedef [SOME_TYPE] T;
typedef [SOME_TYPE] TR;
TR var;
```

the acutal type of var is:

|  T   |  TR  | Type of var |
| :--: | :--: | :---------: |
|  A&  |  T&  |     A&      |
|  A&  | T&&  |     A&      |
| A&&  |  T&  |     A&      |
| A&&  | T&&  |     A&&     |

### const (Keyword)

- const variable: read-only

  > const object cannot call non-const member functions. 
  >
  > member of const object cannot be modified.

- const pointer:

  ```c++
  char greeting[] = "Hello";
  char* p1 = greeting;             // 指针变量，指向字符数组变量
  const char* p2 = greeting;       // 指针变量，指向字符数组常量（const后面是char，说明char只读）
  char* const p3 = greeting;       // 指针常量，指向字符数组变量（const后面是p3，说明p3只读）
  const char* const p4 = greeting; // 指针常量，指向字符数组常量
  ```

- const reference: refrence itself cannot be modified after initialization, so there are only references of const objects, no "const references".

  ```c++
  char c = 'a';
  const int &cr = c;  // compiles
  int &ncr = c;       // won't compile
  const int &n = 5;   // compiles
  int &n = 5          // won't compile
  ```

- const member function:

  - such member function cannot modify member variables. (指针指向的对象是例外)
  - const instance can only call const member functions.
  - const member function cannot be static (static functions are independent on instances)

- const return type: useful when returning a reference of class' internal member

### const variable vs #define

| #define                     | const variable          |
| --------------------------- | ----------------------- |
| text substitution           | a constant variable     |
| handled by preprocessor     | handled by compiler     |
| no type check               | type safe               |
| no memory allcation for it  | memory allocated for it |
| stored in code segment      | stored in data segment  |
| Can be canceled by `#undef` | cannot be canceled      |

### static (keyword)

- static variables
  - normal variables are stored on stack
  - static variables are stored on static segment (scope remains the same), and invisible to other files
- static funcitons: only visible in current file, can be used to avoid conflicts
- static member variables: all instances share one copy, can be accessed without instances 
- static member functions: can be accessed without instances, but cannot use non-static members

> Like `inline`, `static` is only applied at declaration, not in implementation

### Global variable (C++ only, different from C)

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
- 全局静态变量显式用static修饰，作用域是本编译单元，其他编译单元即使用extern声明也不能使用

### extern "C" vs extern "C++"

In C++, when used with a string,`extern` specifies that the linkage conventions of another language are being used for the declarator(s).

Since C++ has **overloading** of function names and C does not, the C++ compiler cannot just use the function name as a unique id to link to, so it mangles the name by adding information about the arguments. `extern "C"` specifies that the function is defined elsewhere and uses the C-language calling convention. 

### #include<> vs #include""

`#include<>`: usually for STL, search in the compiler's include path

`#include""`: first search among the current directory of the source file. if not found, do the `<>` step

Conclusion: for self defined header files, use `#include ""` 

### inline (keyword)

The `inline` keyword suggests the compiler to substitute the code within the function definition for every instance of a function call. Using inline functions can make your program faster because they eliminate the overhead associated with function calls. The compiler can also optimize functions expanded inline in ways that aren't available to normal functions.

Virtual functions can also be inlined, but only when compiler knows the "exact" class of the object. (e.g. non-dynamic resolved, local object, static/global object, etc.)

编译器内部会有一个阈值，在编译一个函数的时候，会计算每条语句所贡献的值（这个值编译器内部有一个映射算法），加完看是否超过，超过就不做inline，不超过就inline。当然这中间还有一些check看看是否能inline之类的。不同的优化选项这个值会不一样，来tradeoff 性能和空间。

inlined functions can have it's implementation at declaration and won't cause re-definition when linking.

### volatile (keyword)

The `volatile` keyword can be applied to variables, in order to prevent the compiler to optimize on it.

### noexcept(keyword)

The `noexcept` operator performs a compile-time check that returns true if an expression is declared to not throw any exceptions.

### explicit (keyword)

The `explicit` keyword is used to mark **constructors** or **conversion functions** to not implicitly convert types in C++. It is optional for **constructors that take exactly one argument** and **conversion functions** since those are the only functions that can be used in typecasting.

### typedef (keyword)

The `typedef` keyword is used for aliasing existing data types, user-defined data types, and pointers to a more meaningful name. Typedefs allow you to give descriptive names to standard data types, which can also help you self-document your code.

```
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

  >Inheritance:
  >
  >for every base class, the derived class has a copy of that class in it's memory. If the base class have virtual member function, there will be a vptr inside that base class. The derived class will have it's own vptr only if it has its own virtual funciton.
  >
  >In case of virtual inheritance: https://www.zhihu.com/tardis/zm/art/380147337?source_id=1003

### cast

- `static_cast`: 这个操作符相当于C语言中的强制类型转换的替代品。多用于非多态类型的转换，比如说将int转化为double。但是不可以将两个无关的类型互相转化。（在编译时期进行转换）

  `static_cast` is used for cases where you basically want to reverse an implicit conversion, with a few restrictions and additions. static_cast performs no runtime checks. This should be used if **you know** that you refer to an object of a specific type, and thus runtime check would be unnecessary.

- `dynamic_cast`: 可以安全的将父类转化为子类，子类转化为父类都是安全的。所以你可以用于安全的将基类转化为继承类，而且可以知道是否成功，如果强制转换的是指针类型，失败会返回 nullptr 指针，如果强制转化的是引用类型，失败会抛出异常。dynamic_cast 转换符只能用于含有虚函数的类, because it uses virtual funciton table to trace super class.

  `dynamic_cast` is useful when **you don't know** what the dynamic type of the object is. It returns a null pointer if the object referred to doesn't contain the type casted to as a base class (when you cast to a reference, a bad_cast exception is thrown in that case).

- `const_cast`: const_cast这个操作符可以去掉变量const属性或者volatile属性的转换符，这样就可以更改const变量了

- `reinterpret_cast`: 重新解释（无理）转换。即要求编译器将两种无关联的类型作转换。仅复制 n 位比特位到d, 不进行任何分析。

### pragma

- `#pragma pack (push)`: Pushes the current packing alignment value on the internal compiler stack.
- `#pragma pack (n)`: Specifies the value, in bytes, to be used for packing.
- `#pragma pack (pop)`: Removes the record from the top of the internal compiler stack.
- `#pragma once`: Specifies that the compiler includes the header file only once when compiling.

### Namespace

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

### operator ::

- `::name`global namespace

- `class-name::member-name` scope is some class

  > can be used to call super class' virtual functions!

- `namespace-name::member-name` scope is some namespace

- Default: nearest scope

### enum

An enumeration is a user-defined type that consists of a set of named integral constants that are known as enumerators.

```c++
namespace CardGame_Scoped
{
    enum class Suit { Diamonds, Hearts, Clubs, Spades };

    void PlayCard(Suit suit) {
        if (suit == Suit::Clubs) // Enumerator must be qualified by enum type
        { /*...*/}
    }
}

namespace CardGame_NonScoped
{
    enum Suit { Diamonds, Hearts, Clubs, Spades };

    void PlayCard(Suit suit) {
        if (suit == Clubs) // Enumerator is visible without qualification
        { /*...*/}
    }
}
```



### Struct vs Class vs Union

| Class                                    | Struct                                   | Union                                    |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| default to private                       | default to public                        | default to public                        |
| can use inheritance                      | can use inheritance                      | no inheritance                           |
| can have member functions, constuctor, and destructor | can have member functions, constuctor, and destructor | can have member functions, constuctor, and destructor |

`struct` is a group of passive data, `class` is for objects that have behavior, and `union` is for very special cases where different data requires to be accessed as different types.



# Class

### Member

##### member initialization

- reference members and const members can only be initialized in construcor initializer list

  > initialization order depends on their order in memory (declaration order)

- static member can only be initialized outside the class

- static const member are not allowed to be initialized at declaration (except for `int`).



### Default Constructor/Copy Constructor/Assignment operator

- If no constructors are declared in a class, the compiler provides an implicit `inline` default constructor. This can be desabled by defining it as `deleted`, or explicitly declared by `default`.

  ```c++
  MyClass() = delete;
  MyOtherClass() = default;
  ```

  > If a class doesn't have a default constructor, it's not possible to create an array of it using brackets `[]` **alone**.

- If no virtual destructors is declared, the compiler provides an default destructor.

- If no move constructor or move-assignment operator is explicitly declared, a copy constructor and a copy-assignment operator will be automatically generated.

- If no copy constructor, copy-assignment operator, move constructor, move-assignment operator, or destructor is explicitly declared, a move constructor and a move-assignment operator will be automatically generated.

- Compiler calls super class's default constuctor implicitly if not called explicitly in program. If super class doesn't have default constructor, then one of the other constructor must be called explicitly.

  ```c++
  Child (): Father() {} // this is an explicit call
  ```

##### Can constructors & destructors be virtual?

No. When constructing the object, the virtual table does not exist. (detailed below) 

Destructors are usually virtual when using inheritance to implement polymorphism.

> when destructor is pure virtual, it must have a implementation out of the class, because the derived class will call it before it call its own destructor.

### Copy constructor & Copy-Assignment operator

```c++
MyClass(const MyClass&);
MyClass& operator= (const MyClass&);
```

Assignment operator cannot be inherited. 

### Move Constructor & Move-Assignment operator

> New in C++11

```c++
MyClass(const MyClass&&);
MyClass& operator= (const MyClass&&);
```

A move constructor enables the resources owned by an rvalue object to be moved into an lvalue without (deep) copying. This is usually faster than copy constuctor/copy-assignment operator when the input is an rvalue! 

> After moving, assign the data members of the source object to default values. This prevents the destructor from freeing resources (such as memory) multiple times.

### Conversion Constructor

Conversion constructor usually refer to constuctor declared without the function specifier `explicit`.

```c++
MyClass(SOME_TYPE st);
```

### Conversion function

Convert object to other type.

```c++
operator SOME_TYPE() const;
```

This function can be called implicitly. The return type of conversion function is implicitly set to "SOME_TYPE". Classes, structures, enumerations, and typedefs cannot be declared within the declaration of the conversion function. (use `explicit` to avoid this)

### Increment operator

```c++
T & operator++(T& t); 			// pre-increase
T & T::operator++(); 			// pre-increase in T's namespace
const T operator++(T& t, int); 	// post-increase
const T T::operator++(int);		// post-increase in T's namespace
```

### Functor

```c++
SOME_TYPE operator()(SOME_TYPE a);
```

### this

当一个对象调用某成员函数时编译器会隐式传入一个参数， 这个参数就是this指针。如果是用new创建的对象可以delete this指针。但是一经delete之后其所有成员都不能再被访问。

### Overload (polymorphism at compile time)

C++ allows multiple definitions for the same function name in the same scope. The definition of the function must differ from each other by the types and/or the number of arguments in the argument list. You cannot overload function declarations that differ only by return type.

### Override (polymorphism at run time)

**Overriding** means the function in derived class implements its own version of the function in base class. There is an optional keyword `override` that can be added to the member function, used to prevent inadvertent inheritance behavior in your code. **Binding** means converting the variable and the function name to address. 

- **Static Binding** (dispatch) is to fix the function address in compile time. C++ is default to static binding.
- **Dynamic Binding** (dispatch) means to decide which function to call based on the dynamic type of the object. In C++, dynamic binding is only possible with virtual functions

### Virtual Function

```c++
class MyClass {
public:
  virtual void virtualFunc() {}
  virtual void pureVirtualFunc() = 0 // no definition, but subclass must implement it
}
```

A virtual function is a member function that you expect to be redefined in derived classes. When you refer to a derived class object using a pointer or a reference to the base class, you can call a virtual function for that object and execute the derived class's version of the function. 

> **Constructor** cannot be virtual, since it's impossible to use a super class pointer to call a child class' constructor. Also, when constucting an instance, vptr has not been initialized, can't find the virtual function.
>
> On the other hand, **destructors** could be, and are often virtual.

> Avoid calling virtual function in constuctors. 当构造函数被调用时，它做的首要的事情之一就是初始化VPTR。然而，它只能知道它属于“当前”类——即构造函数所在的类，完全不知道这个对象是否是基于其它类。构造函数会先调用父类构造函数，此时子类还没有构造，所以此时的对象还是父类的，不会触发多态。(构造和析构期间，virtual函数不是virtual函数)

A **pure virtual function** is a virtual function without implementation. A class have one or more pure virtual funcitons is called **abstract class**. The subclass must implement the pure virtual function.  

Every class that has virtual function(s) has **a virtual function table** constructed at **compile time**, stored at data segment, and accessed by a virtural funciton pointer holding by every instances. The virtual function table contains pointers that pointing at the "nearest" virtual function to it. 

### Inheritance

|                       | public    | protected | private   |
| --------------------- | --------- | --------- | --------- |
| public inheritance    | public    | protected | invisible |
| protected inheritance | protected | protected | invisible |
| private inheritance   | private   | private   | invisible |

### Virtual Inheritance

In multiple inheritance, a class may inherit from a superclass more than once. (think of diamond inheritance graph). Virtual inheritance is a C++ technique that ensures only one copy of a base class's member variables are inherited by grandchild derived classes. In this case, Constructor of repeated superclass **must** be called by the **smallest** subclass' constructor directly, and only once. (An exception is when the subclass has a default empty constructor.)

虚继承的目的是让某个类做出声明，承诺愿意共享它的基类。其中，这个被共享的基类就称为虚基类（Virtual Base Class）。在这种机制下，不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员。

### final

Specifies that a virtual function cannot be overridden or a class cannot be derived from.

### Friend

The`friend` keyword marked some functions or classes as *friends*, granting member-level access to functions and classes.

```c++
class MyClass;
class MyOtherClass;
class MyFriendClass {
public:
  int func(MyClass& o);
}
class MyClass {
    friend class MyOtherClass;
    friend int MyFriendClass::func(MyClass&);
}
```



# Template

##### typename vs class





# Multithreading & Parallelism

### Thread

##### Create/Join/Detach Thread

```c++
#include <thread>
void thread_fun(){};
void thread_fun_para(int x){};
std::thread thread1(thread_fun);
std::thread thread2(thread_fun_para(100));
thread1.join();
thread2.join();
std::thread (thread_fun_para, 100).detach();
```

Code after `join` won't be executed unless thread terminates. If using `detach`, main thread won't wait for child thread to terminate (child thread will be killed if main thread terminates). 

### Mutex

##### mutex

```c++
#include <mutex>
std::mutex m_;
m_.try_lock(); 	// tries to lock the mutex, returns false if the mutex is not available
m_.lock(); 		// lock the mutex, blocks if mutex not available
// protected shared area
m_.unlock();
```

##### shared_mutex

```c++
std::shared_mutex m_;
m_.lock_shared();
m_.unlock_shared();
m.lock();
m.unlock();
```

### Lock

##### lock_guard

`lock_guard`call `mutex.lock()` in constructor, call `mutex.unlock()` in destructor. 

Doesn't support copy and move.

```c++
{
  std::lock_guard<std::mutex> l(mutex_); // lock
  // protected shared area
} // unlock
```

##### unique_lock (write lock)

`unique_lock`call `mutex.lock()` in constructor, call `mutex.unlock()` in destructor. 

More flexible than `lock_guard`, can be moved or copied, provide more interfaces, but use more memory. 

```c++
std::unique_lock<std::mutex> get_lock() { // can be passed
  extern std::mutex m;
  std::unique_lock<std::mutex> l(m);
  // do sth
  return l;  // 不需要 std::move，编译器负责调用移动构造函数
}

int main() {
  std::unique_lock<std::mutex> l(get_lock());
  // do sth
  return 0;
}
```

##### shared_lock (read lock)

##### scoped_lock (C17)

### condition variable

The `condition_variable` class is a synchronization primitive used with a `std::mutex` to block one or more threads until another thread both modifies a shared variable (the *condition*) and notifies the `condition_variable`.

##### Wait

- The first parameter `lock ` **MUST** be an object of type `std::unique_lock<std::mutex>`, which must be locked by the current thread.
- The second parameter `stop_waiting` is a predicate function have a signature of `bool pred()`, which returns false if the waiting should be continued.
- it causes the current thread to block until the condition variable is notified or a spurious wakeup occurs. Atomically unlocks the lock, blocks the current executing thread, and adds it to the list of threads waiting on this cv.

### atomic

##### memory order

- relaxed: not gurantee any thing
- acquire: in current thread, all other "read" must be done after current one
- release: in current thread, all other "write" must be done before current one 
- acq_rel: acquire +. release
- consume:  in current thread, all other "read&write" must be done after current one
- seq_cst: everything happens in the same sequence as in the source code

### async & std::future

`std::async()` The function template async runs the function f asynchronously (potentially in a separate thread which might be a part of a thread pool) and returns a std::future that will eventually hold the result of that function call.

std::async是一个函数模板，会启动一个异步任务，最终返回一个std::future对象。在之前我们都是通过thread去创建一个子线程，但是如果我们要得到这个子线程所返回的结果，那么可能就需要用全局变量或者引用的方法来得到结果，这样或多或少都会不太方便，那么async这个函数就可以将得到的结果保存在future中，然后通过future来获取想要得到的结果。async比起thread来说可以对线程的创建又有了更好的控制，比如可以延迟创建

### for_each & reduce & accumulate





# STL Containers

> All STL containers store a copy of the inserted data!!

### allocator

Dynamically manage memory for stl containers. 考虑到小型区域可能造成内存破碎问题，SGI STL设计了双层级配置器，第一层配置器直接使用malloc()和free()，第二层配置器则视情况采用不同的策略：当配置区块超过128bytes时，调用第一层级配置器，当配置区块小于128bytes时，采用复杂的memory pool方式。

### array (fixed size array)

A `std::array` is a very thin wrapper around a C-style array. It has friendly value semantics, so that it can be passed to or returned from functions by value. Its interface makes it more convenient to find the size, and use with STL-style iterator-based algorithms.

### vector (dynamic array)

##### reserve vs resize

reserve: **only** affect **capacity**, not affact size, won't initialize any instances

Resize: will insert or delete elements to the vector to make it given **size** (could call constructor!)

##### Capacity Growth

The capacity grows by double or 1.5 times of the previous size. Every time a vector's capacity grows the elements need to be copied and all the itrators will becom invalid.

##### erase

erase returns the next iterator of the deteled element. assign this to `it` so that iteration continues. (Map, set won't return next iterator)

### list (doubly linked list)

No random access, only bidirectional iteration.

### Forward_list (linked list)

### deque

In C++, the STL `deque` is a sequential container that provides the functionality of a double-ended queue data structure.

In a regular queue, elements are added from the **rear** and removed from the **front**. However, in a deque, we can insert and remove elements from both the **front** and **rear**. Deque also support random access of its elements.

Internally it maintains a double-ended queue of *chunks* of **fixed size**. Each chunk is a vector, and the queue (“map” in the graphic below) of chunks itself is also a vector.

deque除了具有vector的所有功能外， 还支持高效的首/尾端插入/删除操作，但是占用内存较多

### Priority Queue (heap)

##### Container

The type of the underlying container to use to store the elements. The container must satisfy the requirements of SequenceContainer, and its iterators must satisfy the requirements of LegacyRandomAccessIterator. Additionally, it must provide the following functions with the usual semantics: `front()`, `push_back()`, `pop_back()`. The standard containers std::vector. Also, std::deque satisfy these requirements.

### pair

> \#include <utility>

### Map, Set, Multiset, Multimap (red black tree)

>  set is using a **const iterator** because:
>
>  A set is like a map with no values but only keys. Those keys are used for a tree that accelerates operations on the set, they cannot change. Thus all elements must be const to keep the constraints of the underlying tree from being broken.

### unordered_map, unordered_set (hash map)

### initializer_list

> \#include <initializer_list>

- new in c++11

- used to initialize containers: (because STL containers have corresponding constructors)

  ```c++
  std::map<string, string> const nameToBirthday = {
          {"lisi", "18841011"},
          {"zhangsan", "18850123"},
      };
  ```

- is constructed implicitly when use brace to constuct a list:

  ```c++
  auto s = {1, 2, 3};
  std::list<int> l = {4, 5, 6};
  ```



# STL Funcitons

### move & forward

> introduced in C11

- `std::move` (move semantic, 移动语义) is used to indicate that an object may be "moved from", i.e. allowing the efficient transfer of resources from t to another object. 

- `std::forward<T>` (perfect forwarding, 完美转发) When t is a forwarding reference (a function argument that is declared as an rvalue reference to a cv-unqualified function template parameter), this overload forwards the argument to another function with the value category it had when passed to the calling function. 

  C++11 标准中规定，通常情况下右值引用形式的参数只能接收右值，不能接收左值。但对于函数模板中使用右值引用语法定义的参数来说，它不再遵守这一规定，既可以接收右值，也可以接收左值（此时的右值引用又被称为“万能引用”）。

  ```
  template<class T>
  void wrapper(T&& arg)
  {
      // arg is always lvalue
      foo(std::forward<T>(arg)); // Forward as lvalue or as rvalue, depending on T
  }
  ```

They both do type conversion, implemented using `static_cast`. *Efficient Modern C++* suggests that use `std::move` for rvalue reference and `std:forward` for universal (only in template functions, `&&` reference can take either rvalue or lvalue) reference.

### Stream

A **stream** is a flow of data into or out of a program.

- `<ios>`: Input-Output base classes like `ios_base` and `ios`.

- `<istream>`: contains input stream `istream`; also `iostream` which inherits `istream` and `ostream`.

- `<ostream>`: contains output stream `ostream`

- `<iostream>`:  `cin` is an instance of an `istream`; `cout`, `cerr` and `clog` are instances of `ostream`

  ```c++
  extern istream cin; // so as cout/cerr/clog
  ```

- `<streambuf>`: Stream buffers represent input or output devices and provide a low level interface for unformatted I/O to that device.

- `<fstream>`: contains `ifstream`, `ofstream`, `fsteam` and `filebuf`, which inherit correspondingly `isteam`, `ostream`, `iostream` and `streambuf`. They are used to read/write data from/to files.

- `<sstream>`: contains `istringstream`, `ostringstream`, `stringstream` and `stringbuf`, which inherit `isteam`, `ostream`, `iostream` and `streambuf`, and they provide advanced string operations.

Useful Functions: 

- `getline`: defined in `<string>`, reads a line of characters (separated by `\n` or some other characters) from an input stream and places them into a string.
- `operator>>`: usually read a word (separated by whitespace, `\n`, etc) from stream.



### Files

##### C++ file operations

Stream based, just like `cin` and `cout`.

- ofstream
- ifstream
- fstream

##### C file operations

- fopen & fclose
- fputc & fgetc
- fputs & fgets



# STL Algorithms

> \#include <algorithm>

- swap: quick sort

- find

- sort

  ```
  template< class RandomIt >
  void sort( RandomIt first, RandomIt last );
  // or
  template< class RandomIt, class Compare >
  constexpr void sort( RandomIt first, RandomIt last, Compare comp );
  ```

  a comparator like this needed:

  ```
  bool compare(const MyClass& o1, const MyClass& o2);
  // or 
  
  ```

  

# New Features (C11, C14, C17, C23)

### decltype (specifier)

Inspects the declared type of an entity or the type and value category of an expression.

```c++
struct A { double x; };
const A* a;
 
decltype(a->x) y;       // type of y is double (declared type)
decltype((a->x)) z = y; // type of z is const double& (lvalue expression)
 
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) { // return type depends on template parameters
    return t + u;                       // return type can be deduced since C++14
}
```

> **Tailing Return Type Syntax:**
>
> The trailing return type feature removes a C++ limitation where the return type of a function template cannot be generalized if the return type depends on the types of the function arguments. For example, `a`and `b`are arguments of a function template `multiply(const A &a, const B &b)`, where `a`and `b`are of arbitrary types. Without the trailing return type feature, you cannot declare a return type for the `multiply`function template to generalize all the cases for `a*b`. With this feature, you can specify the return type after the function arguments. This resolves the scoping problem when the return type of a function template depends on the types of the function arguments.

### constexpr (keyword)

The keyword constexpr was introduced in C++11 and improved in C++14. It means constant expression.

Unlike `const`, `constexpr` can also be applied to functions and class constructors. `constexpr` indicates that the value, or return value, is constant and, where possible, is computed at compile time. When a value is computed at compile time instead of run time, it helps your program run faster and use less memory. (low latency!)

when a constexpr function is called with only compile-time arguments, the result of the function will be computed at compile-time. If, however, any argument is notknown at compile-time, the computation will be executed at runtime, like a regular function.

constexpr添加的是顶层const，即如果用在指针上，表示指针本身不能修改，但可以通过指针修改其指向的对象

### auto

The `auto` keyword directs the compiler to deduce the type of a variable from its initialization expression.

- Using auto drops references `&`, `const` qualifiers, and `volatile` qualifiers.
- use `auto` to simplify for iteration
- use `auto` to declare and initialize a variable to a lambda expression.
- Use `auto` together with `decltype`  to write template libraries.

### Smart Pointers

Smart Pointers are used to help ensure that programs are free of memory and resource leaks and are exception-safe. In most cases, when you initialize a raw pointer or resource handle to point to an actual resource, pass the pointer to a smart pointer immediately.

- `unique_ptr`: Allows exactly one owner of the underlying pointer.
- `shared_ptr`: Reference-counted smart pointer. The raw pointer is not deleted until all `shared_ptr` owners have gone out of scope or have otherwise given up ownership. 同一类型的 shared_ptr 智能指针可以相互赋值（拷贝/移动）。
- `weak_ptr`: Special-case smart pointer for use in conjunction with `shared_ptr`. A `weak_ptr` provides access to an object that is owned by one or more `shared_ptr` instances, but does not participate in reference counting.
- `auto_ptr`: deprecated from C17, not recommended to use

Implementation:

- Counting references
- Destructor: when instance is out of scope, the destructor is called automatically

```c++
#include <memory>
```

### lambda expression

A convenient way of defining an anonymous function, full syntax:

`[capture list] (params list) mutable exception-> return type { function body }`

but usually:

- `[capture list] {function body}`

- `[capture list] (param list) {function body}`

- `[capture list] (param list) -> return type {function body}`

  ##### Capture list

  Lambda expression can use the variables not in its scope, but must be included in the capture list `[]`

  pure lambda (non-capturing) expressions are free of side effects, and therefore cannot cause, e.g., race conditions

  

  ```c++
  void abssort(float* x, unsigned n) {
      std::sort(x, x + n,
          // Lambda expression begins
          [](float a, float b) {
              return (std::abs(a) < std::abs(b));
          } // end of lambda expression
      );
  }
  
  // possible to use alias in capture list:
  auto func = [&new_name = this->some_member] () {};
  ```

  

  ```c++
  shared_ptr<vector<Data>> datas;
  auto fun1 = [&]() {
  	//do somthing with datas 100万次
  }
  
  fun1();
  
  auto fun2 = [](shared_ptr<vector<Data>> &d) {
  	//do somthing with d 100万次
  }
  fun2(datas);
  ```


  ### reference qualifier 引用限定 C11

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





  # Compiler 

### C++ compile stages

1. Preprocessing: convert *.cpp/c.* files to *.i* files

   Process`#` instructions: replace `#define`, delete comment, insert `#include` files, etc.

2. Compilation: convert *.i* files to *.s* files

   Translate source code to target assembly code.

3. Assembly: convert *.s* files to *.o* files

   Translate assenbly code to machine instructions (data seg + code seg) and symbol tables. 

   > Only check **declaration**! In symbol table contents: 
   >
   > - symbol name
   > - can/connot referenced by external (static is invisible to other object files)
   > - defined/not defined (not defined variable/func will be checked in linking stage)
   > - location - data/code segment

4. Linking: no specified output name, but default to *a.out*

   Linker takes one or more object files, and combines them to an executable file, a library file, or another object file. Linker checks if, for every object file, every undefined symbol occurs **exactly once** in other object files. 

   > - Stabic linked library: statically included in executable files (linux -- *.a* file; windows -- *.lib* file)
   > - Dynamic linked library: run-time dynamic link(linux -- *.so* file; windows -- *.dll* file)

   > 函数的实现尽量不要放在头文件，不过事实上现在的C++编译器完全可以自动处理类实现写入.h文件的情况，即使实现的成员函数前面是virtual之类不能inline的类型也不会有问题。最多只是降低编译速度而已。

### Libraries

> GCC vs Clang

##### standard C library

*libc* is the earlist standard C library in Linux, and is taken place by *glibc* eventually. 

*glibc* is the GNU's version of standard C library, it's the most popular one and is pre-installed in most Linux systems. *glibc* 实现了 Linux 系统中最底层的 API 库，主要是对系统调用的封装，比如 fopen。同时也提供了一些通用的数据类型和操作，如 string，malloc，signal 等。管理内存分配的函数如 malloc 和 free。 

除此之外还有一些小众 C 库，如用于嵌入式环境的 eglibc，轻量级的 glib 等，它们在 Linux 中不是预装的。

##### standard C++ libraries

*libstdc++* is the standard C++ library of gcc and *libc++* is the standard C++ library of clang.

clang 重写的 libc++ 库比 libstdc++ 更充实，但两者不能兼容。在大多数以 GCC 主导的操作系统中，clang 默认使用的是 gcc 的标准 C++ 库来编译程序，也就是 libstdc++，如果需要使用 libc++，要额外在编译参数中设置。这可能是处于兼容性的考虑。

libstdc++ 是和 gcc 绑定安装的，但 glibc 和 gcc 却没有绑定安装，这是因为 glibc 过于底层，在不同硬件上不能通用，所以绑定安装可能会导致危险的问题，而 libstdc++ 就显得没那么底层了。

##### runtime library

运行时库包含标准库。标准库是程序语言要求的基础功能集合，通常它是独立于不同硬件的，因为语言需要保证一定的可移植性，所以编程语言定出来的库规范，一定是能具有通用性的；但运行时库是需要保障软件在硬件上正常运行的，依据不同的硬件，运行时库的实现可能不同。运行时库对标准库做了扩展，支持软件能够在系统上正常运行。所以，查看标准库规范应该到 C 标准委员会或 C++ 标准委员会的网站上查询，而查看运行时库，需要到对应编译器的手册或运行时库自己的手册中查询。

> 原子操作通常都是硬件强依赖的，所以通常都需要编译器的运行时库来提供支持。

GCC 的运行时库是 libgcc_s，clang 的运行时库是 runtime-rt。如上一节提到的，clang 在大多数 GCC 主导的操作系统中默认使用 GCC 的标准库，同时它也默认使用 GCC 的运行时库。如果需要切换使用 clang 的标准库，那么要额外指定使用 clang 的 runtime-rt 才可以。这需要在编译时给定一些配置参数。当然，你也可以选择把两个版本的运行时库都链接到程序中，但这样通常是冗余和浪费的。

GCC 的运行时库相比 clang 的 runtime-rt，会缺少一些 LLVM 依赖的接口实现。

##### builtin functions

builtin函数是一种在编译器内部实现的，使用它并不需要包含#include<>头文件的支持，在编译的时候，直接变为指令块的替换操作，根据函数的所描述的功能匹配机器文件（MD）中的函数 对应的指令模板，如果找到合适的指令模板就会输出其中的汇编指令，否则就会调用标准库函数。如果编译器内部实现内建函数，那么在从汇编代码生成ELF可执行文件时，就不需要库函数的链接过程。这些内建函数一般是基于不同硬件平台采用专门的硬件指令实现的，因此性能较高。

> GCC compare and swap built-in functions are not supported by clang, for example

### Compiler Flags

##### include directories

Flags specifying the directories in which header files are located, such as "-I" or "-isystem". Ususally stored in environment variable `CPPFLAGS`

##### preprocessor flags

Flags specifying preprocessor definitions, such as "-D". Ususally stored in environment variable `CPPFLAGS`

##### compiler options

Flags that control the behavior of the compiler, such as "-Wall" for enabling all warnings, or "-std=c++11" for specifying the C++ language standard. Ususally stored in environment variable `CFLAGS` or `CXXFLAGS`

##### libraries flags

The names of libraries to be linked, specified using the "-l" flag. Ususally stored in environment variable `LIBS` or `LDLIBS`

> **Order matters here!!**
>
> The linker searches from left to right, and notes unresolved symbols as it goes. If a library resolves the symbol, it takes the object files of that library to resolve the symbol. Dependencies of static libraries against each other work the same - the library that needs symbols **must be first**, then the library that resolves the symbol.
>
> If a static library depends on another library, but the other library again depends on the former library, there is a cycle. You can resolve this by enclosing the cyclically dependent libraries by -( and -), such as -( -la -lb -) (you may need to escape the parens, such as -\( and -\)). The linker then searches those enclosed lib multiple times to ensure cycling dependencies are resolved. Alternatively, you can specify the libraries multiple times, so each is before one another: -la -lb -la.
>
> https://stackoverflow.com/questions/45135/why-does-the-order-in-which-libraries-are-linked-sometimes-cause-errors-in-gcc

##### library directory flags

Library directories: Flags specifying directories in which libraries are located, such as "-L". Ususally stored in environment variable `LDFLAGS`

### Undefined Behavior

##### unsequenced modification and access

> C11: 6.5 Expressions:
>
> If a **side effect** on a scalar object is unsequenced relative to either a different side effect on the same scalar object or a value computation using the value of the same scalar object, the behavior is undefined. If there are multiple allowable orderings of the subexpressions of an expression, the behavior is undefined if such an unsequenced side effect occurs in any of the orderings.
>
> https://blog.csdn.net/qwe_lingkun/article/details/47311417
>
> https://stackoverflow.com/questions/32017561/unsequenced-modification-and-access-to-pointer

Example: `a[i]=a[++i]` , `printf("%d,%d/n", ++i. i++);`

- 不确定点1: 不确定编译器按什么顺序处理表达式，即使大部分编译器从右向左处理（RR parser？）
- 不确定点2: C语言中的++，可能是全部自增结束后依次入栈，也可能是依次先自增入栈再处理下个表达式

