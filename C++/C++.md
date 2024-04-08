this document contains C++ basic topics, or those hard to categorize.

Cpp的复杂性来源：零开销抽象&编译期求值，既要可读性，抽象，又要性能

# Operator & Expressions

- 逗号运算符：取最右边的表达式的值，其他值被丢弃 (`(a, b)`的值是`b`)，优先级最低
- ...
- 括号：Expressions in parenthesis `()` have the highest precedence. parenthesis preserve value, type, and value category.



# const (Keyword)

- const variable: read-only

 > const object cannot call non-const member functions. 
 >
 > member of const object cannot be modified.

- const pointer:

 ```c++
 char greeting[] = "Hello";
 char* p1 = greeting;       // 指针变量，指向字符数组变量
 const char* p2 = greeting;    // 指针变量，指向字符数组常量（const后面是char，说明char只读）
 char* const p3 = greeting;    // 指针常量，指向字符数组变量（const后面是p3，说明p3只读）
 const char* const p4 = greeting; // 指针常量，指向字符数组常量
 ```

- const reference: refrence itself cannot be modified after initialization, so there are only references of const objects, no "const references".

 ```c++
 char c = 'a';
 const int &cr = c; // compiles
 int &ncr = c;    // won't compile
 const int &n = 5;  // compiles
 int &n = 5     // won't compile
 ```

- const member function:
  - such member function cannot modify member variables. (指针指向的对象是例外)
  - const instance can only call const member functions.
  - const member function cannot be static (static functions are independent on instances)

- const return type: useful when returning a reference of class' internal member



# constexpr (C++11)

### constexpr object

`constexpr` 作用于对象时，它本质上就是`const`的加强形式，表明一个值不仅仅是常量，还是编译期可知的。编译期可知的值可能被存放到只读存储空间中。更广泛的应用是“其值编译期可知”的常量整数会出现在需要“整型常量表达式（integral constant expression）的上下文中，这类上下文包括数组大小，整数模板参数，枚举名的值，对齐修饰符（alignas(val)）等。注意 `const`对象不一定会在编译期初始化，因此不一定编译期可知。

### constexpr function

see callable chapter

### constexpr if (C++11?)

???

### constexpr new(C++20)

???



# static (keyword)

- static variables
  - normal variables are stored on stack
  - static variables are stored on static segment (scope remains the same), and invisible to other files

- static funcitons: only visible in current file, can be used to avoid conflicts

- static member variables: all instances share one copy, can be accessed without instances

  > a lot of *inintialization* rules here, please refer to the *initialization* chapeter

- static member functions: can be accessed without instances, but cannot use non-static members



# sizeof

`sizeof` is a compile time operator:

- size of an object of an **empty class** is 1, in order to "ensure that the addresses of two different objects will be different." And the size can be 1 because alignment doesn't matter here, as there is nothing to actually look at.
- Existence of **virtual function(s)** will add 4 bytes of a virtual table pointer in the class. In this case, if the base class of the class already has virtual function(s) either directly or through its base class, then this additional virtual function won't add anything to the size of the class. Virtual table pointer will be common across the class hierarchy.
- Using of **virtual inheritance** will add 4 bytes of a virtual base table pointer in the class. A class will only maintain one virtual base table pointer.
- For classes, only non-static data members will be counted.
- alignment: alignment with the "widest" **basic** type (maybe inside a compound type), and compound types are treated as a whole ({double, char}, char -> 24)
- ordering matters here because of byte padding (char, short, int -> 8 ; char, int, short -> 12)
- char: 1, short: 2, int: 4, double: 8 (depends on GCC version, platform, etc.)






# Namespace

Namespaces provide a method for preventing name conflicts in large projects.

#### operator ::

- `::name` global namespace

- `class-name::member-name` scope is some class

  > can be used to call super class' virtual functions!

- `namespace-name::member-name` scope is some namespace

- Default: nearest scope

### using

- using-directives for namespaces: `using namespace namespace-name;`

  Allowed only in namespace scope and in block scope. Every name from `namespace-name` is visible until the end of the scope in which it appears. 

  > **Don‘t do this in header files!** Including your header will introduces namespace to global namespace, may lead to conflicts! Use *using-declarations* to introduce single member instead.

- using-declarations: `using ns::member;`

  - In namespace and block scope: introduce a member of another namespace into the region
  - In class definition: introduces a member of a base class into the derived class definition
    - introduce base class constructors: `using BaseClassName::BaseClassName;`: For every constructor of BaseClass, compiler generates an identical constructor for derived class.
    - Introduce other functions: `using BaseClassName::FuncName;` for every overload of the function name, generates an identical function for derived class;
  - for enumerators (C++20) `using enum enum-class-name;`

- type alias: refer to types chapter





# Range-based for loop (since C++11)

```c++
for (auto declaration : expression) {
	// loop-statement
}
// is equivalant to:
auto && __range = range-expression;
auto __begin = begin-expr;
auto __end = end-expr;
for ( ; __begin != __end; ++__begin)
{
  range-declaration = *__begin;
  //loop-statement
}
```

- Choose `auto x` when you want to work with copies.
- Choose `auto &x` when you want to work with original items and may modify them.
- Choose `auto const &x` when you want to work with original items and will not modify them.





# static_assert

`static_assert` requires a compile-time predicate, and a message is displayed when the compile-time predicate fails. With C++17, the message is optional. With C++20, this compile-time predicates can be a requires expression. A `static_assert` declaration may appear at namespace and block scope (as a block declaration) and inside a class body (as a member declaration).
