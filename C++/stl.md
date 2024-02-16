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

### Decay 类型退化

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

##### std::alloc 

std::alloc 是SGI STL的默认配置器，它在`<memory>`中实现。他由两级空间配置器实现：

当配置区块超过128bytes时，视之为足够大，便调用第一级配置器，直接使用`malloc()`，`realloc`和`free()`；第二级配置器实现了内存池和自由链表，当配置器区块小于128bytes时，可以从内存池和自由链表中获取空间，减少系统调用，提升性能。（最终都是用`malloc()`，`realloc`和`free()`）



# STL Funcitons

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





# Functionals

### bind

给函数绑定参数变为另一个签名的函数

### function

函数包装器，可包装调用实体如普通函数，函数对象，lamda表达式等：`std::function<int(int)> callback;`

### ref

本质是一个wrapper，可以在使用bind的时候使之变为传引用（默认为传值）





