### C vs C++

- C++ have overloading, reference, class, new/delete
- C is more weakly-typed regarding pointers. Specifically, C allows a `void*` pointer to be assigned to any pointer type without a cast, while C++ does not.


> Effective C++ 条款01: 视C++为一个语言联邦。把C和C++看作两种语言，写代码时需要清楚地知道自己在写C还是C++。如果在写C，请包含头文件`<string.h>`。如果在写C++，请包含`<cstring>`。



# C Standard

`glibc`（GNU C Library）是一个具体的 C 标准库实现，为 GNU/Linux 系统提供标准的 C 库功能。除了 `glibc`，还有其他的 C 标准库实现，例如：musl libc, BSD libc, MSVC CRT 等



# Global variable 

### C++ rules

```c++
int n = 3; // fileA.cpp, definition of non-const global variable
extern n; // fileB.cpp, global variable with external linkage
extern const int i = 4; // fileA.cpp, extern const definition
extern const int i; // fileB.cpp, declaration only 
```

- global variable is defined outside the function
- `extern` specifies that the variable or function is defined in another translation unit. 
- for **non-const** global variables, the `extern` keyword must be applied in all files **except** the one where the variable is defined.
- for **const** global variables, the `extern` keyword should be applied in all files
- The idea of const global variables is to replace the old C style `#define` for constants.

### C rules

todo



# extern "C" vs extern "C++"

In C++, when used with a string,`extern` specifies that the linkage conventions of another language are being used for the declarator(s).

Since C++ has **overloading** of function names and C does not, the C++ compiler cannot just use the function name as a unique id to link to, so it mangles the name by adding information about the arguments. `extern "C"` specifies that the function is defined elsewhere and uses the C-language calling convention. 

There is no real reason to use `extern "C++"`. It merely make explicit the linkage that is the implicit default. If you have a class where some members have extern "C" linkage, you may wish the explicit state that the others are extern "C++".
