# Template

模板能做的事情都是编译期完成的，C++ Template最初是为了提供一个类型安全、易于调试的宏

##### typename vs class

模板参数除了类型外，也可以是一个整型数。这里的整型数比较宽泛，包括布尔型，不同位数、有无符号的整型，甚至包括指针。在模板定义语法中关键字 `class` 与 `typename` 的作用完全一样。其他场景下，`typename` 用于提示后面的字符串为一个类型名称





# (C++11) variadic templates

```c++
template <class... T>
void func(T... args) // args is a template parameter pack
{ //... }
```

### pack expansion

? https://zhuanlan.zhihu.com/p/670867561

### Fold Expressions (C++17)

可以简化对参数包的处理：???似乎是直接就地展开 ??? 和上一个有啥区别？？？

```c++
template<typename T>
string format(T t) {
    std::stringstream ss;
    ss << "[" << t << "]";
    return ss.str();
}

template<typename... Args>
void FormatPrint(Args... args)
{
    (std::cout << ... << format(args)) << std::endl;
}
```

### 继承 variadic template

如下面这个例子，多继承所有模版参数类，并显式启用所有functor。这种场景一般继承的是 lambda（variant 的 visit），如果出现基本类型会无法通过编译。

```c++
template<class... Ts>
struct overloads : Ts... { using Ts::operator()...; };
```





# (C++20) Concept & Requires

specify what's expected of template arguments, and get a nice clear compiler error message if misused

### Usage with integer template parameter

```c++
template <unsigned int i>
requires (i <= 20)         
int sum(int j) { return i + j; }
```

### Concept

```c++
template<typename T>
concept Addable = requires (T a, T b) { a + b; }; // 检查 a 和 b 可加

template<typename T>
concept Printable = requires(T a) { 
    // 检查 T 是否有一个名为print的成员函数，该函数接受一个类型为 std::ostream& 的参数，且返回类型为void
    { a.print(std::cout) } -> std::same_as<void>;
};
```

### Require a concept

```c++
template<typename T>  
requires Addable<T>
auto add(T a, T b) { return a + b; }
// or
template<Printable T> 
void print(const T& t) { t.print(std::cout); }

template<typename T>                                        
requires std::integral<T> // use std concepts whenever possible                
auto gcd(T a, T b) { //... }
```

### Requires anonymous concept

You can define an anonymous concept and directly use it but it should be avoided. Anonymous concepts are less readable and not reuseable.

```c++
template<typename T>
    requires requires (T x) { x + x; } 
auto add(T a, T b) { return a + b; }
```

### logic behind `concept`

```c++
template<typename T> requires std::integral<T>  some_func(const T &a, const T &b) {};
template<class T> concept integral = std::is_integral_v<T>;
template<class _Tp> struct is_integral : _BoolConstant<__is_integral(_Tp)> {};
template<bool _Val> using _BoolConstant _LIBCPP_NODEBUG = integral_constant<bool, _Val>;
```





# Substitution failure is not an error (SFINAE)

Substitution failure is not an error (SFINAE) is a principle in C++ where an invalid substitution of template parameters is not in itself an error. 

当创建一个重载函数的候选集时，某些（或全部）候选函数是用模板实参替换（可能的推导）模板形参的模板实例化结果。如果某个模板的实参替换时失败，编译器将在候选集中删除该模板，而不是当作一个编译错误从而中断编译过程。如果一个或多个候选保留下来，那么函数重载的解析就是成功的，函数调用也是良好的。



# STL helpers

- `is_reference`, `is_lvalue_reference`, `is_rvalue_reference`,`is_const`: 检测顶层const？？？

  ```c++
  template<class T> struct is_array : std::false_type {};
  template<class T> struct is_array<T[]> : std::true_type {};
  template<class T, std::size_t N> struct is_array<T[N]> : std::true_type {};
  ```

- `remove_reference`, `remove_cv`, `remove_const`, `remove_volatile`, ...

  ```c++
  template <class _Tp> struct remove_reference        {typedef _Tp type;};
  template <class _Tp> struct remove_reference<_Tp&>  {typedef _Tp type;};
  template <class _Tp> struct remove_reference<_Tp&&> {typedef _Tp type;};
  template<class T> struct remove_cv { typedef T type; };
  template<class T> struct remove_cv<const T> { typedef T type; };
  template<class T> struct remove_cv<volatile T> { typedef T type; };
  template<class T> struct remove_cv<const volatile T> { typedef T type; };
  ```

- `decay`: remove const, volatile, reference

- `add_pointer`, `add_lvalue_reference`, `add_lvalue_reference`: 

- `enable_if`: if true, have a public member typedef type, otherwise no member

  ```c++
  template<bool B, class T = void> struct enable_if {};
  template<class T> struct enable_if<true, T> { typedef T type; };
  ```

- `conditional`: Provides conditional member `type`, depends on the first boolean template parameter

  ```c++
  template<bool B, class T, class F> struct conditional { using type = T; }; 
  template<class T, class F> struct conditional<false, T, F> { using type = F; };
  ```

- Xxx
