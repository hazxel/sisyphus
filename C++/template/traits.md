# Type traits

注意类型转换尾部的`::type`。如果你在一个模板内部将他们施加到类型形参上，你也需要在它们前面加上`typename`。至于为什么要这么做是因为这些C++11的*type traits*是通过在`struct`内嵌套`typedef`来实现，而正如我之前所说这比别名声明要差。

关于为什么这么实现是因为标准委员会没有及时认识到别名声明是更好的选择，所以直到C++14它们才提供了使用别名声明的版本。这些别名声明有一个通用形式：以 `_t` 结尾，例如`std::transformation<T>::type`在C++14中变成了`std::transformation_t`。

- `is_same<T,U>`: check if `T` and `U` are same types (considering cv-qualifiers)

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

- `underlying_type`: provide the underlying type of `T` if `T` is a complete enum type, otherwise UB

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