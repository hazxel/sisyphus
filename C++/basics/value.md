# Value

Each C++ expression (an operator with its operands, a literal, a variable name, etc.) is characterized by two independent properties: a **type** and a **value category**. 



# Value categories：lvalue vs rvalue (C++11)

##### Primitive categories

- **lvalue** (non-expiring lvalue): 能够用&取地址的表达式，以及字符串字面值（特例）

  > - 具名的对象，包括**具名右值引用**，一定永远是左值。
  >
  >   个人理解：右值只是为了告诉赋值给他的那一方，数据的所有权和生命周期被转交了，但对于接收的那一方，如函数的右值形参，在这个函数执行周期内这个值都是保证不会消亡的，所以其类型实际为 `T&`。
  >
  > - 不具名的对象，可以是左值（左值引用），纯右值（非引用对象），或是将亡值（右值引用）。

- *prvalue* (pure rvalue): 纯右值，即 C++11前的“右值”，包括但不限于：

  - 字符串以外的所有字面值
  - 不具名临时对象如`a+b`, `a++`
  - 返回非引用类型的函数调用，evaluate to 新建对象的表达式，如构造器等

- *xvalue* (expiring value): 将亡值，随着右值引用的引入而新引入。将亡值表达式的形式：

  - 返回右值引用的函数的调用表达式 (`move`, `forward`也算)

##### Mixed categories：

- *glvalue* (generalized left value): inlcudes *lvalue* and *xvalue*
- **rvalue** (right value): includes *prvalue* and *xvalue*



# lifetime extension of temporary objects:

不准确但先这么理解：Temporary objects are rvalue. 就是不具名的临时对象

const lvalue reference`const T&`, rvalue reference`T&&`, and storing by named variable (可能指的是基本类型，不然岂不是会调用构造函数?) are 3 ways to extend the lifetime of a temporary object. **For any statement explicitly binding a reference to a temporary, the lifetime of the temporary would be extended to match the life time of reference.**

Why can‘t temporary bind to non-const reference? C++ doesn't want you to accidentally modify temporaries, because they will die soon. But calling a non-const member function on a modifiable (and non basic typed) rvalue is explicit, so it's allowed.

Historical reason: "lifetime extension of temporary objects" is proposed in 1993, before the existance of RVO. So binding of temporary to a reference will save one copy ctor in such circunstance: 

```c++
Foo bar();						// programming under old C++ standard in 1993
Foo f = bar();				// no (N)RVO back then, copy_ctor called
const Foo &f = bar();	// copy free
Foo f = Foo();				// copy_ctor called, no copy elision back then
const Foo &f = Foo();	// copy free
```

