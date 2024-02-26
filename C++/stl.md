# Components

STL 的目标就是要把数据和算法分开，分别对其进行设计，之后通过 iterator 把这二者再粘接到一起。

- 容器（containers）
- 算法（algorithms）
- 迭代器（iterators）
- 仿函数（functors）协助算法完成各种操作
- 配接器（adapters）用来套接适配仿函数
- 空间配置器（allocator）给容器分配存储空间

必须强调的是，STL 只是一个标准，只对接口进行规范，其实现可以有不同版本。目前流行的 STL实现如 SGI STL 版本被 GCC 采用，此外还有如 Visual C++ 采用的 P.J. Plauger 版本等。



# Traits

Traits 就是为了萃取元素类型而在STL中广泛采用的技法，如`iterator_traits`,  `allocator_traits`, `type_traits` 等，都是为了在编译时进行类型信息的操作，方便在 iterator 或算法中定义中间变量或者返回类型等。以下是简化的 `iterator_traits` 的实现：

```C++
template<class T>
struct iterator_traits {
	typedef typename T::value_type value_type;
};
template <class T> // 原生指针偏特化
struct iterator_traits<T*> {
    typedef T value_type;
};
template <class T> // const 指针偏特化
struct iterator_traits<const T*> {
    typedef T value_type;
};
template <class I> // 这里可以是任意一个算法的实现，比如说取元素的值
typename iterator_traits<I>::value_type
getElement(I ite) { return *ite; }；
```

编译器会询问`iterator_traits<T>::value_type`，若 T 为指针,则进入特化版本,`iterator_traits`直接回答`T`;如果`T`为`class type`,就去询问容器开发者给定的`T::value_type`.



# Decay 类型退化

C++11提供的一个模板类，来为我们移除类型中的一些特性，比如引用、常量、volatile等



# Allocator

STL 容器都带有默认 Allocator 参数：`template<class T, class Allocator = std::allocator<T>`, Allocator 最重要的四个接口为：

- `allocate`：配置空间，足以存储n个T对象
- `deallocate`：释放配置的空间
- `construct`： 调用对象的构造函数
- `destroy`： 调用对象的析构函数

### SGI STL Allocator

##### std::allocator

std::allocator部分符合STL标准，它在文件 defalloc.h 中实现。但是SGI STL的容器并不使用它，也不建议我们使用，它存在的意义仅在于为用户提供一个兼容老代码的折衷方法，其实现仅仅是对new和delete的简单包装。

std::allocators don’t provide a resizing operation, thus may be less performant.

##### std::alloc 

std::alloc 是SGI STL的默认配置器，它在`<memory>`中实现。他由两级空间配置器实现：

当配置区块超过128bytes时，视之为足够大，便调用第一级配置器，直接使用`malloc()`，`realloc`和`free()`；第二级配置器实现了内存池和自由链表，当配置器区块小于128bytes时，可以从内存池和自由链表中获取空间，减少系统调用，提升性能。（最终都是用`malloc()`，`realloc`和`free()`）



# STL I/O

### Stream

A **stream** is a flow of data into or out of a program.

- `<ios>`: Input-Output base classes like `ios_base` and `ios`.

- `<istream>`: contains input stream `istream`; also `iostream` which inherits `istream` and `ostream`.

- `<ostream>`: contains output stream `ostream`

- `<iostream>`: `cin` is an instance of an `istream`; `cout`, `cerr` and `clog` are instances of `ostream`

 ```c++
 extern istream cin; // so as cout/cerr/clog
 ```

- `<streambuf>`: Stream buffers represent input or output devices and provide a low level interface for unformatted I/O to that device.

- `<fstream>`: contains `ifstream`, `ofstream`, `fstream` and `filebuf`, which inherit correspondingly `isteam`, `ostream`, `iostream` and `streambuf`. They are used to read/write data from/to files.

- `<sstream>`: contains `istringstream`, `ostringstream`, `stringstream` and `stringbuf`, which inherit `isteam`, `ostream`, `iostream` and `streambuf`, and they provide advanced string operations.

Useful Functions: 

- `getline`: defined in `<string>`, reads a line of characters (separated by `\n` or some other characters) from an input stream and places them into a string.

  > `getline(ss, str, ',');` can be used to split a string with specified splitter

- `operator>>`: usually read a word (separated by whitespace, `\n`, etc) from stream.

- `flush`: ensures that all data that has been written to that stream is output, including clearing any that may have been buffered.



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



# Functionals

### function

函数包装器，可包装各种类型的调用实体如：普通函数，对象方法，实现了仿函数操作符的对象，lamda表达式等：`std::function<int(int)> callback;`

STL中大量使用function作为算法的入参，如`sort`, `for_each`, `visit` 等

> implementation in-deep: https://zhuanlan.zhihu.com/p/142175297

### bind

给函数绑定参数，使之变为另一个签名的函数

### std::ref

本质是一个wrapper，可以在使用bind的时候使之变为传引用（默认为传值）



# Tuple

原理：继承+variadic templates, 会编译生成参数列表参数个数的类

```c++
template<>struct Tuple<> {};
template<typename Ty1, typename ...Ty2>
struct Tuple<Ty1, Ty2...> : Tuple<Ty2...> {
    Ty1 val;
};
```



# Any (C++17)

`std::any` 的作用是存储任意类型的一段内存，并可以重复赋值，在赋值后可以使用 `std::any_cast` 将其所存储的值转换成特定类型，如果存储的类型与目标类型不匹配，则抛出 `std::bad_any_cast` 异常。

- 可以与模板元编程结合，实现更加灵活的编程技巧。
- 可用于创建泛型容器。这种容器可以存储任意类型的对象，而不需要在编译期间知道这些对象的具体类型。例如`std::vector<std::any>`



# Variant (C++17)

union在许多性能敏感场景下被使用，但它没有办法推断自己当前使用的类型，析构函数也不能被正常调用，而variant提供了一种类型安全的union类型。如果你正在处理一些底层的逻辑，并且只使用基本类型，那么union可能仍然是首选。但是对于其他的使用场景，std::variant是一种更好的方式。

一些可能使用的场景：

- 所有可能为单个变量获得几种类型的地方：解析命令行、ini文件、语言解析器等。
- 有效地表达计算的几种可能结果：例如求解方程的根
- 错误处理：例如，您可以返回`variant<Object, ErrorCode>`，如果返回值是有效的，则返回`Object`，否则分配一些错误码（C++23可以使用`std::expected`）。
- 状态机
- 不使用虚表和继承实现的多态（visiting pattern）

https://zhuanlan.zhihu.com/p/607734474

### get

Variant cannot be directly used, but via non-member function:

- `std::get<Type|Index>(variant)`: return a reference or throw a `std::bad_variant_access`
- `std::get_if<Type|Index>(variant)`: return a pointer or`nullptr`, won't throw exception.

### visit

它允许我们对 `std::variant` 进行模式匹配，根据其存储的类型执行不同的操作。可 visit 多个 variant。

### monostate

使用 `std::monostate`表示“无值”或“空”状态的 `std::variant`。
