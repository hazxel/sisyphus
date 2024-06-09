# cast

### Implicit conversion

合法的隐式类型转换场景：

- 安全转换：
  - 不会导致精度丢失或数据溢出的基本类型转换
  - 派生类对象转换为基类对象，可能导致对象切片 (object sclicing problem)
  - 派生类指针转换为基类指针
  - 基类引用绑定派生类对象
  - 添加 cv-qualifier 的转换 (e.g. non-const to const)
  - `T[]` to `T*`
  - arithmatic (`int`, `double`, ...) to `bool`: 0 -> `false`, other -> `true`
  - bool to arithmetic: `true` -> 1, `false` -> 0
  - pointers(`T*` or `T[]`) to `bool`: `nullptr`/`NULL` -> `false` , other -> true
  - function type to "pointer to that function"
- 窄化转换：指**基本类型**之间的转换中，由于目标类型的表示范围小于源类型导致的精度丢失或数据溢出。合法，但会触发编译器警告。（小知识：函数重载需要转换`float`时，C++在`int`和`double`重载都存在时倾向于选择`double`）
- non-`explicit` conversion function, non-`explicit` convertion constructor

### explicit (keyword)  -  forbid implicit conversions

The `explicit` keyword forbid **implicit conversions** and **copy-initialization** for **constructors** and **conversion functions**. (those are the only functions that used in type-casting)

> in C++20 `explicit` can be used with an constant expression, e.g. `explicit (...)`, which take effects only if expression evaluate to `true`

### old style cast (parenthesis `()`)

C style 强制类型转换, should be replaced with modern C++ style casts.

### static_cast

相当于C语言中的强制类型转换的替代品。多用于**非多态类型**的转换，比如说将 `int` 转化为 `double`。但是不可以将两个无关的类型互相转化。（在编译时期进行转换）不能包含底层const

 `static_cast` is used for cases where you basically want to reverse an implicit conversion, with a few restrictions and additions. static_cast performs no runtime checks. This should be used if **you know** that you refer to an object of a specific type, and thus a check would be unnecessary.

### dynamic_cast

`dynamic_cast` 操作符只能用于含有虚函数的类 (uses virtual funciton table to trace super class)，可以安全的将父类的**指针或引用**和子类**指针或引用**相互转化。转换失败时，如果转换的是指针类型，会返回 `nullptr` ，如果是引用类型，会抛出 `std::bad_cast` 异常。

`dynamic_cast` is useful when **you don't know** what the dynamic type of the object is.

### const_cast

`const_cast` 操作符可以去掉变量的 `const` 属性或者 `volatile` 属性的转换符，这样就可以更改const变量了

### reinterpret_cast

重新解释（无理）转换。即要求编译器将两种无关联的类型作转换。

why should we **AVOID** reinterpret_cast: [weblink here](https://blog.hiebl.cc/posts/practical-type-punning-in-cpp/)

C++ compiler is allowed to assume that when de-referenced, two pointers of incompatible types do not have the same value (i.e. do not point to the same chunk of memory). By using `reinterpret_cast` you break the compiler’s assumption, leading to undefined behavior.

```c++
int init_vars(int* a, float* b) {
   *a = 42;
   *b = 0.f;
   return *a; // the compiler is allowed to assume 42 is returned
}

int main() {
   int a_and_b{12};
   // the compiler will optimize this to `return 42`
   return init_vars(&a_and_b, reinterpret_cast<float*>(&a_and_b));
}
```

### bit_cast(C++20): ???

similar to reinterpret cast, but **MUCH** safer. Only allow conversion between closely related types that shares the same underlying memory layout. (e.g. int to float)



# Runtime Type Information（RTTI）

In C++, RTTI can be used to do safe typecasts using the `dynamic_cast<>` operator, and to manipulate type information at runtime using the `typeid` operator and `std::type_info` class.

C++引入这个机制是为了让程序在运行时能根据基类的指针或引用来获得该指针或引用所指的对象的实际类型。但是现在RTTI的类型识别已经不限于此了，它还能通过typeid操作符识别出所有的基本类型（int，指针等）的变量对应的类型。相关操作：

1. typeid运算符，该运算符返回其表达式或类型名的实际类型
2. dynamic_cast运算符，该运算符将基类的指针或引用安全地转换为派生类类型的指针或引用

### macro 开关

RTTI可被编译时的宏开关启用或关闭，如 STL 源码中常见的`__cpp_rtti`, `_LIBCPP_NO_RTTI` 等，也可通过形如`-fno-rtti`的指令关闭，有些编译器默认关闭RTTI以消除性能开销。启用RTTI时，vtable 布局中会增加一个 slot 用于存放 typeinfo 指针。Without RTTI you can't use typeid, dynamic_cast, and some STL classes are compiled differently.

### typeid (operator)

运算符`typeid(n)`中，参数`n`可以是类型、变量、字面量等。如果参数类型没有虚函数，就直接在编译期完成运算。如果参数类型有虚函数，延迟等到运行期间才能确定值（读虚表）。

### type_info (class)

`std::type_info`  is the class returned by the `typeid` operator. It holds the **name** of the type and means to compare two types for equality or collating order(可能作为key放入map，需要比较顺序). 

`std::type_info` 对象在编译期生成，编译器会在静态存储空间 .data 段里为这些 `type_info` 对象分配空间，并初始化它们的内容。遵循 Itanium C++ ABI 的编译器（例如GCC和Clang）对 `type_info` 的初始化，本质上等同于在全局作用域里加入如下 C++ 代码：`type_info _ZTI3Foo("Foo");`, 然后根据ABI要求，将指向这些 `type_info` 对象的指针放进 `vtable` 。



# Type alias

### Using

- type alias: `using identifier = type-id;`

  Similar to `typedef`, introduces a name which is a synonym for the type denoted by `type-id`. If used inside of a class or sturct, can introduce **member type alias**.

- alias template (C++11): `template<T> using identifier = type-id;`

  Like any template declaration, can only be declared at class or namespace scope.

### typedef

The `typedef` is a C style keyword help to aliasing types, help to give a more meaningful name to a type.

```c++
typedef std::vector<std::vector<int>> MatrixInt;
```

If used inside of a class or sturct, can define **member type alias**.

### typedef vs using - using is always better

- `using` is compatible with templates, `typedef` is not. (typedef alias need to be a concrete type)
- `typedef` sometimes result in *dependent type* (type that dependent on template param`T`). When using dependent type, template tell if `T::A` is a type or is a member of T, so `typename` needed.
- 在复杂类型如函数，成员指针等的定义时，二者在语法上的的相似之处在于 `using`  就是把`typedef` 中的名字挪到前面再加上 `=`



# cv type qualifier???

### const

顶层const？？？top-level cv-qualified？？？

### volatile

The `volatile` keyword can be applied to variables, in order to prevent the compiler to optimize on it. 通常用于驱动程序的开发中

### mutable



# incomplete types

简而言之，就是指存在声明，但没有定义的类型，如：

- void
- arrays of unspecified length
- structures and unions with unspecified content
