# Class

### Member Init order

成员变量初始化的顺序为：先进行声明时初始化，然后进行初始化列表初始化，最后进行构造函数初始化

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

- 如果 没有定义 拷贝构造/拷贝赋值/移动构造/移动赋值/析构 函数的任何一个，编译器会 自动生成 移动构造/移动赋值 函数（rule of zero）
  如果 需要定义 拷贝构造/拷贝赋值/移动构造/移动赋值/析构 函数的任何一个，不要忘了 移动构造/移动赋值 函数，否则对象会 不可移动（rule of five）
  尽量使用=default 让编译器生成 移动构造/移动赋值 函数，否则 容易写错

- Compiler calls super class's default constuctor implicitly if not called explicitly in program. If super class doesn't have default constructor, then one of the other constructor must be called explicitly.

 ```c++
 Child (): Father() {} // this is an explicit call
 ```

### Copy constructor & Copy-Assignment operator

```c++
MyClass(const MyClass&);
MyClass& operator= (const MyClass&);
```

Assignment operator cannot be inherited. 

### Move Constructor & Move-Assignment operator (C++11)

```c++
MyClass(const MyClass&&);
MyClass& operator= (const MyClass&&);
```

A move constructor enables the resources owned by an rvalue object to be moved into an lvalue without (deep) copying. This is usually faster than copy constuctor/copy-assignment operator when the input is an rvalue.

如果需要自定义 移动构造/移动赋值 函数，尽量定义为 `noexcept` 不抛出异常（编译器生成的版本会自动添加），否则 **不能高效使用标准库和语言工具**！例如，标准库容器 `std::vector` 在扩容时，会通过 `std::vector::reserve()` 重新分配空间，并转移已有元素。如果扩容失败，`std::vector` 满足强异常保证 (strong exception guarantee)，可以回滚到失败前的状态。为此，`std::vector` 使用 `std::move_if_noexcept()` 进行元素的转移操作。如果 没有定义移动构造函数 或 自定义的移动构造函数没有 `noexcept`，会导致 `std::vector` 扩容时执行无用的拷贝，不易发现。

### Conversion Constructor

Conversion constructor usually refer to constuctor declared without the function specifier `explicit`.

```c++
MyClass(SOME_TYPE st);
```

### Conversion function (C++11)

```c++
operator SOME_TYPE() const; // no arguments
```

Conversion functions can be called implicitly. May lead to ambiguity sometimes if the receiver class also has a conversioin constructor.

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

使类能像函数一样被调用，好处是可以存放状态

### this

当一个对象调用某成员函数时编译器会隐式传入一个参数， 这个参数就是this指针。如果是用new创建的对象可以delete this指针。但是一经delete之后其所有成员都不能再被访问。

### Destructor

##### private dtor？

make dtor private will force the class to be only created by `new`. Similarly, overload with private `new` and `delete` will force the class to be only created on stack.

### Overload (polymorphism at compile time) 重载

C++ allows multiple definitions for the same function name in the same scope. The definition of the function must differ from each other by the types and/or the number of arguments in the argument list. You cannot overload function declarations that differ only by return type.

### Override (polymorphism at run time) 重写

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

Every class that has virtual function(s) has **a virtual function table** constructed at **compile time**. It is accessed by a virtural funciton pointer holding by every instances. The virtual function table contains pointers that pointing at the "nearest" virtual function to it.



### Inheritance

default is private

|            | public  | protected | private  |
| --------------------- | --------- | --------- | --------- |
| public inheritance  | public  | protected | invisible |
| protected inheritance | protected | protected | invisible |
| private inheritance  | private  | private  | invisible |

##### Virtual Inheritance

In multiple inheritance, a class may inherit from a superclass more than once. (think of diamond inheritance graph). Virtual inheritance is a C++ technique that ensures only one copy of a base class's member variables are inherited by grandchild derived classes. In this case, Constructor of repeated superclass **must** be called by the **smallest** subclass' constructor directly, and only once. (An exception is when the subclass has a default empty constructor.)

虚继承的目的是让某个类做出声明，承诺愿意共享它的基类。其中，这个被共享的基类就称为虚基类（Virtual Base Class）。在这种机制下，不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员。

##### using 

用于引入基类中的成员函数或成员变量到派生类中，主要有以下几种场景：

- 子类无法隐式使用父类的构造函数，需要写`Derived(int arg): Base(int arg) {}` 略麻烦
- 子类的同名函数会隐藏基类的实现（即使是参数列表不同的重载）

##### final

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



### Mutable

某些const函数内需要改变成员变量值，但又要保持const属性，被const对象调用。用mutable修饰成员变量，在const成员函数中也可修改。



# Struct vs Class vs Union

| Class                         | Struct                        | Union                         |
| ----------------------------------------------------- | ----------------------------------------------------- | ----------------------------------------------------- |
| default to private                  | default to public                   | default to public                   |
| can use inheritance                  | can use inheritance                  | no inheritance                    |
| can have member functions, constuctor, and destructor | can have member functions, constuctor, and destructor | can have member functions, constuctor, and destructor |

`struct` is a group of passive data, `class` is for objects that have behavior, and `union` is for very special cases where different data requires to be accessed as different types.



# Enum

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



# Object Slicing

将 subclass 的对象赋值到 baseclass 对象时，或使用 baseclass 的拷贝构造函数拷贝派生类时，将发生对象切片：基类副本将没有在派生类中定义的任何成员变量。切片问题是 C++98 中默认按值传递名声不好的重要原因。