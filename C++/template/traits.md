Traits 就是为了萃取元素类型而在STL中广泛采用的技法，如`iterator_traits`,  `allocator_traits`, `type_traits` 等，都是为了在编译时进行类型信息的操作，方便在 iterator 或算法中定义中间变量或者返回类型等。



# Type Traits

注意类型转换尾部的`::type`。如果你在一个模板内部将他们施加到类型形参上，你也需要在它们前面加上`typename`。至于为什么要这么做是因为这些C++11的*type traits*是通过在`struct`内嵌套`typedef`来实现，而正如我之前所说这比别名声明要差。

关于为什么这么实现是因为标准委员会没有及时认识到别名声明是更好的选择，所以直到C++14它们才提供了使用别名声明的版本。这些别名声明有一个通用形式：以 `_t` 结尾，例如`std::transformation<T>::type`在C++14中变成了`std::transformation_t`。

- `is_same<T,U>`: check if `T` and `U` are same types (considering cv-qualifiers)

- `is_empty<T>`: check if `T` is an empty type (that is, a non-union class type with no non-static data members other than bit-fields of size 0, no virtual functions, no virtual base classes, and no non-empty base classes) 

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

- `result_of`/`invoke_result`: deduces type of an invoke expression of a callable at compile time



# Iterator Traits

以下是简化的 `iterator_traits` 的实现：

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