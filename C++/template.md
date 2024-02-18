# Template

模板能做的事情都是编译期完成的，C++ Template最初是为了提供一个类型安全、易于调试的宏

##### typename vs class

模板参数除了类型外，也可以是一个整型数。这里的整型数比较宽泛，包括布尔型，不同位数、有无符号的整型，甚至包括指针。在模板定义语法中关键字 `class` 与 `typename` 的作用完全一样。其他场景下，`typename` 用于提示后面的字符串为一个类型名称





### (C++11) variadic templates

```c++
template <class... T>
void func(T... args) // args is a template parameter pack
{ //... }
```

##### pack expansion

? https://zhuanlan.zhihu.com/p/670867561

##### Fold Expressions (C++17)

可以简化对参数包的处理：???似乎是直接就地展开

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



### (C++20) Concept & Requires

specify what's expected of template arguments, and get a nice clear compiler error message if misused

```c++
template<typename T> requires std::integral<T>  some_func(const T &a, const T &b) {};
template<class T> concept integral = std::is_integral_v<T>;
template<class _Tp> struct is_integral : _BoolConstant<__is_integral(_Tp)> {};
template<bool _Val> using _BoolConstant _LIBCPP_NODEBUG = integral_constant<bool, _Val>;
```

