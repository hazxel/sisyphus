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
- 窄化转换：指**基本类型**之间的转换中，由于目标类型的表示范围小于源类型导致的精度丢失或数据溢出。合法，但会触发编译器警告。
- non-`explicit` conversion function, non-`explicit` convertion constructor

### explicit (keyword)  -  forbid implicit conversions

The `explicit` keyword forbid **implicit conversions** and **copy-initialization** for **constructors** and **conversion functions**. (those are the only functions that used in type-casting)

> in C++20 `explicit` can be used with an constant expression, e.g. `explicit (...)`, which take effects only if expression evaluate to `true`

### parenthesis cast

???

### static_cast

相当于C语言中的强制类型转换的替代品。多用于**非多态类型**的转换，比如说将int转化为double。但是不可以将两个无关的类型互相转化。（在编译时期进行转换）不能包含底层const

 `static_cast` is used for cases where you basically want to reverse an implicit conversion, with a few restrictions and additions. static_cast performs no runtime checks. This should be used if **you know** that you refer to an object of a specific type, and thus a check would be unnecessary.

### dynamic_cast

可以安全的将父类转化为子类，子类转化为父类都是安全的。所以你可以用于安全的将基类转化为继承类，而且可以知道是否成功，如果强制转换的是指针类型，失败会返回NULL指针，如果强制转化的是引用类型，失败会抛出异常。dynamic_cast 转换符只能用于含有虚函数的类, because it uses virtual funciton table to trace super class.

 `dynamic_cast` is useful when **you don't know** what the dynamic type of the object is. It returns a null pointer if the object referred to doesn't contain the type casted to as a base class (when you cast to a reference, a bad_cast exception is thrown in that case).

### const_cast

const_cast这个操作符可以去掉变量const属性或者volatile属性的转换符，这样就可以更改const变量了

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

RTTI可被编译时的宏开关启用或关闭，如 STL 源码中常见的`__cpp_rtti`, `_LIBCPP_NO_RTTI` 等，可通过形如`-fno-rtti`的指令关闭。有些编译器默认关闭RTTI以消除性能开销。But usually without RTTI you can't use typeid, dynamic_cast, and some STL classes are compiled differently.

### type_info

The class type_info holds the **name** of the type and means to compare two types for equality or collating order(有时需要比较顺序因为可能作为key放入map). This is the class returned by the typeid operator.

std::type_info对象是在编译的时候决定其内容的，作为静态数据存在于最终生成的目标代码里。编译器会在静态存储空间里为这些type_info对象分配空间，并生成代码来初始化它们的内容。对于遵循Itanium C++ ABI的编译器（例如GCC和Clang）来说，其中编译器给生成的初始化type_info的代码，本质上就跟自己在全局作用域里写个这样的C++代码类似 `type_info _ZTI3Foo("Foo");`, 然后根据ABI要求，将指向这些type_info对象的指针放进vtable即可。



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



# cv type qualifier???

### const

顶层const？？？top-level cv-qualified？？？

### volatile

The `volatile` keyword can be applied to variables, in order to prevent the compiler to optimize on it. 通常用于驱动程序的开发中

### mutable
