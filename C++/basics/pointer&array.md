# Pointer

### operations

- 指针加减整数：等效为 `ptr + i * sizeof(T)`, 步长是被指向的元素的大小。
- 指针之间相减：必须是相同类型指针，可计算两个指针之间的元素个数，等效为 `(p-q)/sizeof(T)`
- 指针的乘除法和指针之间相加都是非法运算
- `&` 取地址(address of operator)
- `*` 解引用(dereference)

### NULL vs nullptr

`nullptr` is a compile time constant, with type `nullptr_t`, and can be implicitly converted to any pointer type. `nullptr` resolves the ambiguity of `NULL`, because it's hard to tell if `NULL` is pointer or number. **Always** prefer `nullptr` in C++!

```c++
#define NULL ((void*)0) // definition in C
#define NULL 0			// definition in C++
typedef decltype(nullptr) nullptr_t;
```

### void pointer

Can point to any kinds of variable. Some compiler forbid arithmatic (+/-) operation to void pointers. **C++ does not allow implicit conversion** of `void*`, while C allows.

### pointer to function

```c++
// function declaration
void f(int);
class Widget {
  static void f(int); // Widget::f has the same type as free function f
}

// parenthesis matters here, otherwise * and & will be treated as if with return type
typedef void FuncPtr(int); // function pointer alias
typedef void (*FuncPtr)(int); // function pointer alias
typedef void (&FuncRef)(int); // function reference alias
using FuncPtr = void(int); // function pointer alias
using FuncPtr = void(*)(int); // function pointer alias
using FuncRef = void(&)(int); // function reference alias

void (*funcPtr)(int) = f; // function pointer variable 
void (&funcRef)(int) = f; // function reference variable

(*funcPtr)(123); // function call via function pointer
funcRef(123);		 // function call via function reference
```

### pointer to member

Pointer to non-static data member, and pointer to non-static member function:

```c++
class Widget {
  int getVal() { return val; }
  int val;
}

typedef int Widget::*DataMemberPtr;
using DataMemberPtr = int Widget::*;
typedef int(Widget::*MemberFuncPtr)(int);
using MemberFuncPtr = int(Widget::*)(int);

int Widget::*pVal = &Widget::val; // ptr to data member
DataMemberPtr pVal = &Widget::val; // equivalent
int (Widget::*pGetVal)() = &Widget::getVal; // ptr to non-statics member function
MemberFuncPtr pGetVal = &Widget::getVal; // equivalent

Widget w;
w.*pVal = 2; 						// access data member with instance
cout << (w.*pGetVal)(); // call member function with instance

Widget *pW = &w;
pW->*pVal = 3;						// access data member with pointer
cout << (pW->*pGetVal)(); // call member function with pointer
```

> There is no reference-to-member type in C++



# Array

- must be init with a brace-enclosed initializer

 ```c++
int arr[10] = {};
int mat[][2] = {{1,2}, {3,4}}; // first level size can be omitted
int arr2[10] = arr;	// won't compile, must use {}
 ```

- `[]` dereference the variable just as `*`

 ```c++
arr[5] = 0;
*(arr+5) = 0;		// equivalent
mat[1][1] = 4; 	// a[n][m] 表示 *(*(a+n)+m)
 ```

- implicitly convert to pointer

 ```c++
int *ptrInt = arr;		
int (*ptrMat)[2] = mat;	// only consider the first layer
 ```

- `+` and `-`: step size is `sizeof()` element type, 和指针区别是

- `*` and `&`: actual type of pointer and reference

 ```c++
int (*parr)[10] = &arr;	
int (&rarr)[10] = arr;
 ```

- 只能在数组定义所在的代码区中获得数组长度(in bytes)

 ```c++
int size = sizeof(arr);	// 10 * sizeof(int)
auto foo = [](int[] arr) { 	// won't get the size of the array
  std::cout << sizeof(arr) << std::endl; // sizeof(int*)
} 	// size of array doesn't follow when passed to function
 ```

- will call ctor and dtor automatically for non-built-in types:

 ```c++
struct Foo {
  int x;
  Foo() = delete;
  Foo(int) {}
  Foo(int,int) {}
}
Foo a[100];		// won't compile, need to call DC() but deleted
Foo b[2]{1,2}; 	// call DC(int) twice
Foo c[1]{{1,2}};	// call DC(int,int) once
 ```



