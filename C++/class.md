# Class

##### Member Init order

成员变量初始化的顺序为：先进行声明时初始化，然后进行初始化列表初始化，最后进行构造函数初始化

##### Default Constructor/Copy Constructor/Assignment operator

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

##### Copy constructor & Copy-Assignment operator

```c++
MyClass(const MyClass&);
MyClass& operator= (const MyClass&);
```

Assignment operator cannot be inherited. 

##### Move Constructor & Move-Assignment operator

> New in C++11

```c++
MyClass(const MyClass&&);
MyClass& operator= (const MyClass&&);
```

A move constructor enables the resources owned by an rvalue object to be moved into an lvalue without (deep) copying. This is usually faster than copy constuctor/copy-assignment operator when the input is an rvalue! 

> After moving, assign the data members of the source object to default values. This prevents the destructor from freeing resources (such as memory) multiple times.

##### Conversion Constructor

Conversion constructor usually refer to constuctor declared without the function specifier `explicit`.

```c++
MyClass(SOME_TYPE st);
```

##### Conversion function

Convert object to other type.

```c++
operator SOME_TYPE() const;
```

This function can be called implicitly. The return type of conversion function is implicitly set to "SOME_TYPE". Classes, structures, enumerations, and typedefs cannot be declared within the declaration of the conversion function.

> May lead to ambiguity sometimes if the receiver class also has a conversioin constructor.

##### Increment operator

```c++
T & operator++(T& t); 			// pre-increase
T & T::operator++(); 			// pre-increase in T's namespace
const T operator++(T& t, int); 	// post-increase
const T T::operator++(int);		// post-increase in T's namespace
```

##### Functor

```c++
SOME_TYPE operator()(SOME_TYPE a);
```

使类能像函数一样被调用，好处是可以存放状态

##### this

当一个对象调用某成员函数时编译器会隐式传入一个参数， 这个参数就是this指针。如果是用new创建的对象可以delete this指针。但是一经delete之后其所有成员都不能再被访问。

##### Destructor

##### Can constructors be virtual?

No. When constructing the object, the virtual table does not exist.

##### Overload (polymorphism at compile time)

C++ allows multiple definitions for the same function name in the same scope. The definition of the function must differ from each other by the types and/or the number of arguments in the argument list. You cannot overload function declarations that differ only by return type.

##### Override (polymorphism at run time)

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

##### Runtime Type Information（RTTI）

In C++, RTTI can be used to do safe typecasts using the dynamic_cast<> operator, and to manipulate type information at runtime using the typeid operator and std::type_info class.

RTTI是Runtime Type Identification的缩写，意思是运行时类型识别。C++引入这个机制是为了让程序在运行时能根据基类的指针或引用来获得该指针或引用所指的对象的实际类型。但是现在RTTI的类型识别已经不限于此了，它还能通过typeid操作符识别出所有的基本类型（int，指针等）的变量对应的类型。

C++通过以下的两个操作提供RTTI：

1. typeid运算符，该运算符返回其表达式或类型名的实际类型
2. dynamic_cast运算符，该运算符将基类的指针或引用安全地转换为派生类类型的指针或引用



### List Initialization (C++11)

使用大括号 `{}` 来指定初始值，可以用于变量、数组、结构体、类等，他不允许窄化转换（不会隐式转换丧失精度），并且可读性更好。如果有构造函数接受一个 `std::initializer_list` 参数，列表初始化会优先选择这个构造函数。初始化对象是一个类的情况下需要满足：

- 没有用户声明的构造函数
- 没有用户提供的构造函数(允许显示预置或弃置的构造函数)
- 没有私有或保护的非静态数据成员
- 没有基类
- 没有虚函数
- 没有`{}`和`=`直接初始化的非静态数据成员
- 没有默认成员初始化器



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
- 子类的同名函数会隐藏基类的实现（即使是重载）

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