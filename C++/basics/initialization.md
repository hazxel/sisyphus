# Initialization

### Basic Syntax

```c++
T declarator();								// direct init
T declarator{};								// list init
T declarator = <expression>;	// copy init
T declarator = {};						// possible
```

- `{}` doesn't allow narrowing conversion (good thing), also called uniform initialization
- `()` can't use no-param-ctor (*most vexing parse* : `Widget w();` initialization or function declaration?)
- Non-copyable ojbects cannot use `=` to initialize

### List Initialization (C++11)

##### Aggregate Initialization

Aggregate Initialization is a special case of List Initialization. 用花括号初始值列表初始化聚合类(Aggregate)数据成员，初始值的顺序必须与声明的顺序一致，若初始值列表的元素个数少于类的成员数量，则靠后的成员被值初始化。聚合类型定义为：

- 普通数组，如int[5]，char[]

- 满足下列所有条件的 `class` , `struct` 或 `union`：

 - 没有用户提供的构造函数 (允许 `default` 或 `delete`)

 - 没有私有或保护的非静态数据成员

 - 没有虚函数
 - 如果有继承关系，必须是公开、非虚继承 (C++17)

注意基类是否是聚合类型与派生类是否为聚合类型没有关系。

标准库 `<type_traits>` 中提供了 `is_aggregate`，可帮助判断目标类型是否为聚合类型。

> GCC 支持小括号初始化聚合类型，允许窄化转换，其他编译器不支持

##### For non-aggregate types

大括号初始化大部分时候是优于小括号初始化的。它能被用于各种不同的上下文，防止隐式窄化。若对象没有包含`std::initializer_list`形参的构造函数，花括号初始化和圆括号初始化效果完全相同。编译器看到 `{t1, t2, …, tn}`便会构造一个 `initializer_list`，它关联到一个 `array<T, n>`，该array内的元素会被编译器分解并逐一传给构造函数。

但当对象拥有接收`std::initializer_list`的构造函数时，大括号初始化的语义比较混乱，例如：

```c++
std::vector<int> v1{5, 6}; // 2 elems: {5, 6}
std::vector<int> v2(5, 6); // 5 elems: {6, 6, 6, 6, 6}
```

这种情况下，花括号初始化会先尝试将实参转换为一个最匹配的`initializer_list<P>`，然后去尝试匹配形参`initializer_list<T>`，逻辑如下：
- 如果`T`不能转换为 `P` ，编译器会考虑其他构造函数
- 如果`T`是基本类型且可以转换为 `P` ，即使存在类型匹配更准确的非`std::initializer_list`形参构造函数，也会被忽略
  - 如果`T`需要窄化才可转换为`P`，编译器就会报错
  - 如果`T`不需窄化就可转换为`P`， 编译器就会选择该函数

想传入一个空的 `std::initializer_list`, 必须要这么写：`Widget w({}); `。此外，在模板中使用花括号构造对象时情况会更麻烦，因为模版类通常不知道关于模版参数类是否有包含一个`std::initializer_list`形参的构造器。(e.g. `std::make_unique`, STL 的解决方案是使用圆括号，并记录至接口文档中)

##### STL initializer_list

`std::initializer_list` is an array of `const T` objects. It will be automatically constructed when:

- a *brace-init-list* is the right operand of `=` 
- Pass a *brace-init-list* to a function, and the function accepts `std::initializer_list` parameter
- Use a *brace-init-list* to construct an object and exists a ctor accepts `std::initializer_list` （窄化转换会编译失败）
- a *brace-init-list* is bound to auto

所有的标准容器的构造函数都有以initializer_list为参数的构造函数。initizlizer_list的最广泛的使用就是不定长度同类型参数的情况。

##### Designated Initializers(C++20)

？？？



### initialize static data member

`static` data members must be defined in **exactly one** translation unit to fulfil the one definition rule. So its definition is ususally in .cpp file, to avoid multiple definition in each translation unit that includes the header file.

`const static`, and non-`volatile` data member of integral or enumeration types can be initialized with a `initializer` in class. Other non-trivial initialization need to call Ctor, which could invoke all sorts of functions that might not be available on initialisation.

`constexpr static` data member of LiteralType must be initialized with an initializer in which every expression is a constant expression, right inside the class definition. A `constexpr static` data member is implicitly `inline`.

Since C++17, a `static` data member may be declared `inline` in class.

