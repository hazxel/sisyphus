# Member Init order

成员变量初始化的顺序为：先进行声明时初始化，然后进行初始化列表初始化，最后进行构造函数初始化



# this

当一个对象调用某成员函数时编译器会隐式传入一个参数， 这个参数就是 `this` 指针。如果是用 `new` 创建的对象可以`delete this`指针。但是一经`delete`之后其所有成员都不能再被访问。



# special member functions

1. Default Constructor (DftCtor) `Widget();` : 

   Implicit definition of DftCtor: empty body, empty initializer list, calls DftCtor of base classes, and of non-static members. ??? what if they don't have DftCtor? or is basic type?

2. Copy constructor (CCtor) `Widget(const Widget&);` 默认为逐成员拷贝

3. Copy-Assignment operator (CAssOp) `Widget& operator= (const Widget&);` 默认为逐成员拷贝

4. Move Constructor (MCtor) `Widget(const Widget&&);` 默认为逐成员移动

5. Move-Assignment operator (MAssOp) `Widget& operator= (const Widget&&);` 默认为逐成员移动

   > 如果需要自定义 移动构造/移动赋值 函数，尽量定义为 `noexcept` 不抛出异常（编译器生成的版本会自动添加），否则 **不能高效使用标准库和语言工具**！例如，标准库容器 `std::vector` 在扩容时，会通过 `std::vector::reserve()` 重新分配空间，并转移已有元素。如果扩容失败，`std::vector` 满足强异常保证 (strong exception guarantee)，可以回滚到失败前的状态。为此，`std::vector` 使用 `std::move_if_noexcept()` 进行元素的转移操作。如果 没有定义移动构造函数 或 自定义的移动构造函数没有 `noexcept`，会导致 `std::vector` 扩容时执行无用的拷贝，不易发现。

6. Destructor (Dtor) `~Widget();`

   默认生成的版本：默认为`noexcept`，且是否为 `virtual` 与基类(如果有)的析构函数一致

   > make dtor private/delete will force the class to be only created by `new`. Similarly, overload with private `new` and `delete` will force the class to be only created on stack.

### Implicit definition of special memeber functions & Rule of 3/5/0

Copmiler implicitly defines a default inline version for the 6 special member functions except:

- If exists any kind of custom Ctor, DftCtor won't be implicitly defined.
- If any of Dtor, CCtor, CAssOp, **MCtor** or **MAssOp** is defined by user, **MCtor** and **MAssOp** won't be implicitly defined. 此时如果不定义移动构造/移动赋值，对象会不可移动。移动构造和移动赋值，如果你声明了其中一个，编译器就不再生成另一个。
- If MCtor or MAssOp is defined by user, CCtor and CAssOp will be implicitly defined as **DELETED**!!

> 注意：1. **成员函数模版**不会阻止编译器生成特殊成员函数。2. `=delete`&`=default`也属于用户定义实现

这么设计的核心逻辑就是，如果资源管理很简单，编译器就用 trival 的方法代替你来实现；反之你如果定义了析构函数，拷贝或者移动，编译器就觉得你是要自己实现复杂的管理，默认的实现多半不适用了

**Rule of Three**: If a class requires a custom Dtor, a custom CCtor, or a custom CAssOp, it almost certainly requires all three. (Otherwise may lead to incorrect management of resources)

**Rule of Five**: Because a custom Dtor, CCtor or CAssOp prevents the  implicit definition of the MCtor and the MAssOp, Rule of Three can expand to Rule of Five if move semantics are desirable. (Otherwise lose potential optimization of move)

**Rule of Zero**: Classes that have custom Dtors, CCtor, MCtor, CAssOp or MAssOp should deal exclusively with ownership (which follows from the Single Responsibility Principle). Other classes should not have any custom versions of those five. (C++ Core Guidelines - C.20: Avoid defining default operations if possible.)

### Delete and default

Only special member functions can be `default`. All functions can be `delete`, even non-member ones.

deleted 函数不能被使用，但还是会被纳入重载决议。这样的好处是可以在编译期拒绝一些不合适的函数调用。

 ```c++
MyClass() = delete;
MyOtherClass() = default;
 ```

尽量使用 `default` , 增强可读性，且防止偶然定义阻止了 MCtor 或 MAssOp 的生成。

尽量使用 `delete` 代替 undefined private, 前者报错信息更明确，且后者有时还是会延迟到链接期才报错（成员函数或友元的调用）。此外还应将 `delete`的函数声明为`public`，防止错误信息被 `private` 错误覆盖

 > If a class doesn't have a default constructor, it's not possible to create an array of it using brackets `[]` **alone**.



# Conversion 

### Conversion constructor

Conversion constructor usually refer to constuctor declared without the function specifier `explicit`.

```c++
MyClass(SOME_TYPE st);
```

### Conversion function (C++11)

```c++
operator SOME_TYPE() const; // no arguments
```

Conversion functions can be called implicitly. May lead to ambiguity sometimes if the receiver class also has a conversioin constructor.



# Operator

### Increment operator

```c++
T & operator++(T& t); 			// pre-increase
T & T::operator++(); 			// pre-increase in T's namespace
const T operator++(T& t, int); 	// post-increase
const T T::operator++(int);		// post-increase in T's namespace
```

### `->` operator

???



# Functor

```c++
SOME_TYPE operator()(SOME_TYPE a);
```

使类能像函数一样被调用，好处是可以存放状态



### Overload (polymorphism at compile time) 重载

C++ allows multiple definitions for the same function name in the same scope. The definition of the function must differ from each other by the types and/or the number of arguments in the argument list. You cannot overload function declarations that differ only by return type.

### Override (polymorphism at run time) 重写

**Overriding** means the function in derived class implements its own version of the function in base class. There is an optional keyword `override` that can be added to the member function, used to prevent inadvertent inheritance behavior in your code. **Binding** means converting the variable and the function name to address. 

- **Static Binding** (dispatch) is to fix the function address in compile time. C++ is default to static binding.
- **Dynamic Binding** (dispatch) means to decide which function to call based on the dynamic type of the object. In C++, dynamic binding is only possible with virtual functions



# Inheritance

default specifier is private.

|            | public  | protected | private  |
| --------------------- | --------- | --------- | --------- |
| public inheritance  | public  | protected | invisible |
| protected inheritance | protected | protected | invisible |
| private inheritance  | private  | private  | invisible |

##### Virtual Inheritance

In multiple inheritance, a class may inherit from a superclass more than once. (think of diamond inheritance graph). Virtual inheritance ensures only one copy of a base class's member variables are inherited by grandchild derived classes. In Virtual Inheritance, Ctor of repeated superclass **must** be called by the **smallest** subclass' Ctor directly, and only once. (An exception is when the subclass has a default empty constructor.)

虚继承就是让某个类做出声明，承诺愿意共享它的基类，被共享的基类就称为虚基类（Virtual Base Class）。在这种机制下，不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员。

##### using 

用于引入基类中的成员函数或成员变量到派生类中，主要有以下几种场景：

- 子类无法隐式使用父类的构造函数，需要写`Derived(int arg): Base(int arg) {}` 略麻烦
- 子类的同名函数会隐藏基类的实现（即使是参数列表不同的重载）

但引入 base class 成员函数时只能指定名字并引入所有同名重载

##### final

Specifies that a virtual function cannot be overridden or a class cannot be derived from.

##### Special Member

- Constructor are not inherited. (but still possible via `using Base::Base;`)
- If derived class doesn't call base class DftCtor explicitly (i.e.`Child():Father(){}`), compiler will call it implicitly. If super class doesn't have DftCtor, then an other Ctor must be called explicitly.  
- Assignment operators are not inherited. 



# Virtual Function

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

C++规范并没有规定虚函数的实现方式，但大部分编译器都用虚函数表（vtable）来实现。不同编译器对单继承的实现都比较接近；而多继承可能需要多层虚函数表（vtable table, VTT），实现的差异会比较大。Every class that has virtual function(s) has **a virtual function table** constructed at **compile time**. It is accessed by a virtural funciton pointer holding by every instances. The virtual function table contains pointers that pointing at the "nearest" virtual function to it.



# Friend???

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



### Mutable

某些const函数内需要改变成员变量值，但又要保持const属性，被const对象调用。用mutable修饰成员变量，在const成员函数中也可修改。



# Syntax for non-static member functions

### cv-qualifier???

 ### ref-qualifier 引用限定 (C++11)

成员函数默认不区分左值和右值 receiver。如果想要区分，采用以下语法：

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



# Struct vs Class vs Union

| Class                         | Struct                        | Union                         |
| ----------------------------------------------------- | ----------------------------------------------------- | ----------------------------------------------------- |
| member default to private           | member default to public            | member default to public            |
| can inheritance                  | can inheritance                  | no inheritance                    |
| can have member functions, constuctor, and destructor | can have member functions, constuctor, and destructor | can have member functions, constuctor, and destructor |

`struct` is a group of passive data, `class` is for objects that have behavior, and `union` is for very special cases where different data requires to be accessed as different types.



# Enum

An enumeration is a user-defined type that consists of a set of named constants known as enumerators.

```c++
// Scoped enum, C++!1
enum class Suit { Diamonds, Hearts, Clubs, Spades };
void PlayCard(Suit suit) {
  if (suit == Suit::Clubs) { /*...*/ }
}
// NonScoped, C++98
enum Suit { Diamonds, Hearts, Clubs, Spades };
void PlayCard(Suit suit) {
  if (suit == Clubs) { /*...*/ }
}
```

Scoped enum is usually better because it:

- reduce name conflicts
- no implicit conversion, always need cast
- 限域`enum`总是可以前置声明，因为他们的默认底层类型是 `int`。非限域`enum`由于出现较早，仅当指定它们的底层类型时才能前置。不能前置声明最大的缺点就是可能增加编译依赖（改动头文件触发大量重新编译）
- 至少有一种情况下非限域`enum`是很有用的：使用`std::tuple`时替换`std::get<1>`使之更有可读性



# Object Slicing

将 subclass 的对象赋值到 baseclass 对象时，或使用 baseclass 的拷贝构造函数拷贝派生类时，将发生对象切片：基类副本将没有在派生类中定义的任何成员变量。切片问题是 C++98 中默认按值传递名声不好的重要原因。