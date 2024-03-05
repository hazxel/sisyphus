# lambda

A convenient way of defining an anonymous function, full syntax:

`[capture list] (params list) mutable exception-> return type { function body }`

but usually:

- `[capture list] {function body}`

- `[capture list] (param list) {function body}`

- `[capture list] (param list) -> return type {function body}`

 ### Capture list

 Lambda expression can use the "outer" variables in it's scope, but must be included in the capture list `[]`

 pure lambda (non-capturing) expressions are free of side effects, and therefore cannot cause, e.g., race conditions 

捕获列表会形成一个闭包，实现原理呢就是靠语法糖生成一个匿名的结构体，捕获的都会作为这个匿名结构体中的变量

 ```c++
void abssort(float* x, unsigned n) {
  std::sort(x, x + n,
    // Lambda expression begins
    [](float a, float b) {
      return (std::abs(a) < std::abs(b));
    } // end of lambda expression
  );
}
 ```

  

 ```c++
shared_ptr<vector<Data>> data;
auto fun1 = [&]() {
	//do somthing with data 1 million times
};
fun1();
// 使用捕获列表时，会多一次内存寻址：需要先把被捕获的对象的地址存在栈上，再把该栈上存地址的单元的地址作为入参传入，同样的，在函数内访问该捕获对象时也需要取地址两次
 
auto fun2 = [](shared_ptr<vector<Data>> &d) {
	//do somthing with data 1 million times
};
// 通过参数传入引用时，和普通函数无区别
fun2(data);
 ```

> 在使用 gcc 编译时，捕获列表为空，似乎也还是会安排一个字节的空间到栈上占位置:
>
> The current object (*this) can be implicitly captured if either capture default is present. If implicitly captured, it is always captured by reference, even if the capture default is `=`. The implicit capture of *this when the capture default is `=` is deprecated.(since C++20)

### generic lambda (C++14)

generic lambda has `auto` in its parameter list, it's equivalent to:

```c++
auto lambda = [](auto x, auto y) {return x + y;};

struct unnamed_lambda
{
  template<typename T, typename U>
    auto operator()(T x, U y) const {return x + y;}
};
auto lambda = unnamed_lambda();
```

？？？泛型闭包：

```c++
auto f3 = [](auto a) {
  return [=]() mutable { return a = a + a; };
};
auto twice1 = f3(1);
cout << twice1() << endl; // 2
cout << twice1() << endl; // 4
auto twice2 = f3(string{"a"});
cout << twice2() << endl; // aa
cout << twice2() << endl; // aaaa
```





# STL Functionals

### function

函数包装器，可包装各种类型的调用实体如：普通函数，对象方法，实现了仿函数操作符的对象，lamda表达式等：`std::function<int(int)> callback;`

STL中大量使用function作为算法的入参，如`sort`, `for_each`, `visit` 等

> implementation in-deep: https://zhuanlan.zhihu.com/p/142175297

### bind

给函数绑定参数，使之变为另一个签名的函数

### std::ref

本质是一个wrapper，可以在使用bind的时候使之变为传引用（默认为传值）
