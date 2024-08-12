# Data Member Init order

成员变量初始化的顺序为：先进行声明时初始化，然后进行初始化列表初始化(**按类定义的顺序，而不是初始化列表中的顺序**)，最后进行构造函数初始化



# special member functions

1. Default Constructor (DftCtor) `Widget();` : 

   Implicit definition of DftCtor: empty body, empty initializer list, calls DftCtor of base classes, and of non-static members. ??? what if they don't have DftCtor? or is basic type?

   > It's not possible to create an array of a class without default constructor using brackets `[]` **alone**.

2. Copy constructor (CCtor) `Widget(const Widget&);` 默认为逐成员拷贝

3. Copy-Assignment operator (CAssOp) `Widget& operator= (const Widget&);` 默认为逐成员拷贝

   > 注意 `Widget w2 = w1; ` 调用的是 CCtor 而不是 CAssOp：

4. Move Constructor (MCtor) `Widget(const Widget&&);` 默认为逐成员移动

5. Move-Assignment operator (MAssOp) `Widget& operator= (const Widget&&);` 默认为逐成员移动

   > 如果需要自定义 移动构造/移动赋值 函数，尽量定义为 `noexcept` 不抛出异常（编译器生成的版本会自动添加），否则 **不能高效使用标准库和语言工具**！

6. Destructor (Dtor) `~Widget();`

   默认生成的版本：默认为`noexcept`，且是否为 `virtual` 与基类(如果有)的析构函数一致

   > make dtor private/delete will force the class to be only created by `new`. Similarly, overload with private `new` and `delete` will force the class to be only created on stack.

> 自定义的 CCtor, CAssOp, MCtor, MAssOp 的第一个形参的 cv-qualifier 组合可以是任意的，甚至可以在第一个参数后添加任意个有默认值的其他参数。（e.g. `Widget(Widget& w, int num = 1);` 是一个合法的 CCtor）此外，CCtor, CAssOp, MCtor, MAssOp 中的任意一个都可以有多个重载，编译器会自己选取最佳匹配。

### Implicit definition of special memeber functions & Rule of 3/5/0

Copmiler implicitly defines a default inline version for the 6 special member functions except:

1. DftCtor won't be implicitly defined if exists any kind of custom Ctor.
2. **MCtor** and **MAssOp** won't be implicitly defined if any of Dtor, CCtor, CAssOp, **MCtor** or **MAssOp** is defined by user.
3. CCtor and CAssOp will be implicitly defined as **DELETED** if MCtor or MAssOp is defined by user.

> 注意：1. **成员函数模版**不会阻止编译器生成特殊成员函数。2. `=delete`&`=default`也属于用户定义实现 3. 有一个说法是自定义的 Dtor, CCtor, CAssOp 会阻碍生成默认的 Dtor, CCtor, CAssOp，但这个说法好像是错误的

这么设计的核心逻辑就是，如果资源管理很简单，编译器就用 trivial 的方法代替你来实现；反之你如果定义了析构函数，拷贝或者移动，编译器就觉得你是要自己实现复杂的管理，默认的实现多半不适用了

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

Conversion functions can be called implicitly if without specifier `explicit`. May lead to ambiguity sometimes if the receiver class also has a conversioin constructor.

### auto conversion (C++14)

```c++
operator auto() {}
```





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

##### this

当一个对象调用某成员函数时编译器会隐式传入一个参数， 这个参数就是 `this` 指针。如果是用 `new` 创建的对象可以`delete this`指针。但是一经`delete`之后其所有成员都不能再被访问。

### Function call operator

```c++
SOME_TYPE operator()(SOME_TYPE a);
```

使类能像函数一样被调用 (Functor)，相比普通函数好处是可以存放状态



# Virtual Function

### Binding (dispatch)

Binding means converting the variable and the function name to address. 

- **Static Binding**: fix the function address in compile time. C++ is default to static binding.
- **Dynamic Binding**: decide which function to call based on the dynamic type of the object. In C++, dynamic binding is only possible with virtual functions.

### Virtual

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

### Override  (重写)

**Overriding** means the function in derived class implements its own version of the function in base class. To override, must fulfill: 

- base class's method is`virtual`
- Names are same (except for Dtor)
- Parameter lists are same
- `const`ness are same
- exception specifications are same

- (C++11) *reference qualifiers* are same

永远为重写函数加上`override`, 避免想要重写的时候因为某些东西不匹配而重写失败



# Syntax for non-static member functions

### cv-qualifier???

`const` 类型实例只能调用 `const` 修饰的成员函数

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

### Mutable

用`mutable`修饰成员变量后，在`const`修饰的成员函数中也可被修改。

使用场景为：某些`const`函数内需要改变成员变量值，但又要保持`const`属性以便被`const`对象调用。注意此时需要确保`const`成员函数是线程安全的，因为大家会默认多线程并发读操作是安全的，除非你**确定**此函数永远不会在并发上下文（*concurrent context*）中使用。



# Inheritance

default specifier is private.

|                       | public    | protected | private   |
| --------------------- | --------- | --------- | --------- |
| public inheritance    | public    | protected | invisible |
| protected inheritance | protected | protected | invisible |
| private inheritance   | private   | private   | invisible |

### Virtual Inheritance

In multiple inheritance, a class may inherit from a superclass more than once. (think of diamond inheritance graph). Virtual inheritance ensures only one copy of a base class's member variables are inherited by grandchild derived classes. In Virtual Inheritance, Ctor of repeated superclass **must** be called by the **smallest** subclass' Ctor directly, and only once. (An exception is when the subclass has a default empty constructor.)

虚继承就是让某个类做出声明，承诺愿意共享它的基类，被共享的基类就称为虚基类（Virtual Base Class）。在这种机制下，不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员。

### `final` keyword

Specifies that a virtual function cannot be overridden or a class cannot be derived from.

### `using` keyword

用于引入基类中的成员函数或成员变量到派生类中，主要有以下几种场景：

- 子类不会隐式继承使用父类的构造函数，需要写`Derived(int arg): Base(int arg) {}` 略麻烦
- 子类的同名函数会隐藏基类的实现（即使是参数列表不同的重载）

但引入 base class 成员函数时只能指定名字并引入所有同名重载

### Special Member in Inheritance

- Constructor are not inherited. (but still possible via `using Base::Base;`)
- If derived class doesn't call base class DftCtor explicitly (i.e.`Child():Father(){}`), compiler will call it implicitly. If super class doesn't have DftCtor, then an other Ctor must be called explicitly.  
- Assignment operators (CAssOp/MAssOp) are not implicitly inherited. 
- Dtor is not implicitly inherited



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



# Polymorphism 多态

### Object Slicing

将 subclass 的对象赋值到 baseclass 对象时，或使用 baseclass 的拷贝构造函数拷贝派生类时，将发生对象切片：基类副本将没有在派生类中定义的任何成员变量。切片问题是 C++98 中默认按值传递名声不好的重要原因。

### 实现多态的方式

- 通过对象：不能通过对象实现多态。由于先前提到的对象切片问题，通过对象调用成员函数时，不触发多态，因为不仅无法访问派生类的成员，还额外引入了虚表跳转开销。因此编译器直接在编译期绑定函数地址。
- 通过引用：虽然引用可以实现多态，但一般也不通过引用实现多态，因为不如指针用起来灵活：
  - 引用只能在初始化时绑定对象，一经绑定便不能修改
  - 引用不是对象，不能创建引用的数组
- 使用指针实现多态是最常用的方法，强大且危险。