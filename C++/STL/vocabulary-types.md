# Optional & Expected

manages an optional contained value. Common use is return value for a function that may fail.

### optional

表达“有时存在”或“暂不存在”。可以在不采用异常机制的情况下，反映一个 no-value 的 case。比如轻松区分到底是有返回值还是没有，而不是用傻傻的"magic value"。

### expected

The class template `std::expected` provides a way to store either of two values, an *expected* value of type `T`, or an *unexpected* value of type `E`. `std::expected` is never valueless.



# pair

> \#include <utility>

存的是拷贝

### Special member function

- CopyCtor & MoveCtor: `defaulted`
- CopyAssignOperator is `deleted` if either type's `is_copy_assignable<T>::value` evaluates to false. 
- MoveAssignOperator is `deleted` if either type's `is_move_assignable<T>::value` evaluates to false. 





# Tuple

存的是拷贝

Implementation：继承+variadic templates, 会编译生成参数列表参数个数的类

```c++
template<>struct Tuple<> {};
template<typename Ty1, typename ...Ty2>
struct Tuple<Ty1, Ty2...> : Tuple<Ty2...> {
    Ty1 val;
};
```



# Any (C++17)

`std::any` 的作用是存储任意类型的一段内存，并可以重复赋值(所以`any`不是模版类，而是有个模版化的构造/拷贝/析构函数)，在赋值后可以使用 `std::any_cast` 将其所存储的值转换成特定类型，如果存储的类型与目标类型不匹配，则抛出 `std::bad_any_cast` 异常。

- 可以与模板元编程结合，实现更加灵活的编程技巧。
- 可用于创建泛型容器。这种容器可以存储任意类型的对象，而不需要在编译期间知道这些对象的具体类型。例如`std::vector<std::any>`

实现：

基本都需要使用 `std::type_info` 来判断类型

- 使用 lambda 来记忆类型信息

  ```c++
  struct any
  {
      void* data_;
      std::type_info const& (*getType_)();
      void* (*clone_)(void* otherData);
      void (*destroy_)(void* data);
  
      template<typename T>
      explicit any(T&& value)
          : data_{new T{std::forward<T>(value)}}
          , getType_{[]() -> std::type_info const&{ return typeid(T); }}
          , clone_([](void* otherData) -> void* { return new T(*static_cast<T*>(otherData)); })
          , destroy_([](void* data_) { delete static_cast<T*>(data_); }) {}
  
      any(any const& other)； // copy ctor， omitted
      ~any() { destroy_(data_); }
  };
  ```

  

- MSVC：内部保存了 `std::type_info` 的指针。将内存分为 Trivial、Small、Big 三种，Trivial 内存直接对拷，Small 内存需要保存额外的拷贝、移动、销毁指针，具体操作是 in_place 的，Big 内存需要保存额外的拷贝、销毁指针，具体操作是堆内存的 new、delete。

- GCC: 使用 `any::_S_manager` 模版类，区分 internal和external 存储？？？



# Variant (C++17)

union在许多性能敏感场景下被使用，但它没有办法推断自己当前使用的类型，析构函数也不能被正常调用，而variant提供了一种类型安全的union类型。如果你正在处理一些底层的逻辑，并且只使用基本类型，那么union可能仍然是首选。但是对于其他的使用场景，std::variant是一种更好的方式。

一些可能使用的场景：

- 所有可能为单个变量获得几种类型的地方：解析命令行、ini文件、语言解析器等。
- 有效地表达计算的几种可能结果：例如求解方程的根
- 错误处理：例如，您可以返回`variant<Object, ErrorCode>`，如果返回值是有效的，则返回`Object`，否则分配一些错误码（C++23可以使用`std::expected`）。
- 状态机
- 不使用虚表和继承实现的多态（visiting pattern）

https://zhuanlan.zhihu.com/p/607734474

### Constructor

- 对于默认构造，会用第一个类型进行初始化。但如果第一个类型没有默认构造函数时将会初始化失败。

  > `std::monostate`类型表示“无值”或“空”状态。将variant的第一种类型指定为`std::monostate`以解决第一个类型没有默认构造函数导致 variant 不能默认构造的问题。

- 对于大部份场景，可以不指定类型，如 `variant<float, std::string> v{10.5f};`

- 有歧义场景需要指定类型 `variant<long, float> v{in_place_index<1>, 7.6};` 因为这里是 double

### get

Variant cannot be directly used, but via **non-member function**:

- `std::get<Type|Index>(variant)`: return a reference or throw a `std::bad_variant_access`
- `std::get_if<Type|Index>(variant)`: return a pointer or`nullptr`, won't throw exception.

> Why non-member function? If `get<T>()` was made a member function template, a `template` keyword would be needed when it is called in a dependent context. (In ADL, disambiguate whether `get` is a member function template or variable followed by operator `<`)  For example:
>
> ```cpp
> template <typename Variant>
> void f(Variant const& v) {
>  auto x0 = v.template get<T>(); // if it were a member
>  auto x1 = get<T>(v);           // using a non-member function
> }
> ```

### visit

对 `std::variant` 进行模式匹配，根据其存储的类型执行不同的操作。codes speak for themselves：

##### Naive Usage: provide visit funcition for every types via a visitor class

```c++
struct MultiplyVisitor {
    float mFactor;
    MultiplyVisitor(float factor) : mFactor(factor) { }
    void operator()(int& i) const { i *= static_cast<int>(mFactor); }
    void operator()(float& f) const { f *= mFactor; }
    void operator()(std::string& ) const { // nothing to do here...}
};
  
std::variant<int, float, std::string> intFloatString{1};
std::visit(MultiplyVisitor(0.5f), intFloatString);
```

##### provide callable for every types via lambda and a helper class

```c++
template<class... Ts>
struct overloads : Ts... { using Ts::operator()...; };
 
int main() {
    std::variant<int, std::string> var1{42}, var2{"abc"};
    auto use_int = [](int i){ std::cout << "int = " << i << '\n'; };
    auto use_str = [](std::string s){ std::cout << "string = " << s << '\n'; };
#if (__cpp_lib_variant >= 202306L)
    var1.visit(overloads{use_int, use_str});
    var2.visit(overloads{use_int, use_str});
#else // not a member function before C++23
    std::visit(overloads{use_int, use_str}, var1);
    std::visit(overloads{use_int, use_str}, var2);
#endif
}
```

##### generic lambda to generate callable set

```c++
auto PrintVisitor = [](const auto& t) { std::cout << t << "\n"; };
std::variant<int, float, std::string> intFloatString { "Hello" };
std::visit(PrintVisitor, intFloatString);
```

##### 还可一次 visit 多个 variant

？？？ to be added



# in_place (C++17)

`std::in_place`, `std::in_place_type`, `std::in_place_index` 都是用于 **消除歧义** 的标签，为了配合 C++17 新引入的几个数据结构 `std::expected`, `std::optional`, `std::variant`, `std::any`

### in_place

`in_place`可以传递给 `std::option`,`std::expected` 表示 **原位** 使用 `T` 的构造函数构造对象，而不是使用 `optional` 或 `expected` 的默认构造函数。

```c++
std::optional<Big> o1{std::in_place, "1"}; // Big{"1"}
std::optional<Big> o2{std::in_place};      // Big{}
std::optional<Big> o3{};                   // optional()
```

### in_place_tpye & in_place_index

 `std::in_place_type`, `std::in_place_index` 可以传递给  `variant`, `any`，指明应构造何种类型的对象。

```c++
using variant_t = std::variant<std::string, std::vector<int>>
variant_t v1{std::in_place_type<std::string>, 4, 'A'} // string{4,'A'}
variant_t v2{std::in_place_index<1>, 4, 42} 					// vector<int>(4,42)
```

