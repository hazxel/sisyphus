

# Pre-processor

### #include<> vs #include""

`#include<>`: usually for STL, search in the compiler's include path

`#include""`: first search among the current directory of the source file. if not found, do the `<>` step

Conclusion: for self defined header files, use `#include ""` 

### #include_next

#include_next 也是包含一个头文件，但它的语义是包含“指定的这个文件所在的路径的后面路径的那个文件”。

假设`#include` 的搜索路径的顺序依次是A，B，C，D。在B目录中有 a.h，在D目录中也有 a.h，如果 `#include <a.h>`，会包含 B 目录中的 a.h ，但如果 `#include_next <a.h>`，就会包含 D 中的a.h。预处理器在B目录中搜索到a.h头文件后，会以B目录作为分割点，搜索B目录后面的目录（C，D），然后在这后面的目录中搜索a.h头文件，并把在这之后搜索到的a.h头文件包含进来。



# pragma

**avoid using `pragma`, it's not part of C++ standard, it's merely compiler extension**

- `#pragma pack` family:
  - `#pragma pack (n)`: Specifies the value, in bytes, to be used for packing. (default value: 8) 会影响文件中所有后续的结构体定义，直到遇到 `#pragma pack(pop)` 指令
  - `#pragma pack (push)`: Pushes the current alignment value to the internal compiler stack.
  - `#pragma pack (push, n)`: Pushes the current alignment value, set `n` as current alignment value
  - `#pragma pack (pop)`: Pop from the compiler stack and set it as the current alignment value.
  - `#pragma pack(pop, n)`: Removes the record from the top, set `n` as current alignment value
  - `#pragma pack(show)`: throw a compile time warning to indicate current alignment value

- `#pragma once`: Specifies that the compiler includes the header file only once when compiling.



# Macro

### Built-in Macros

- `__FILE__`: expand to the full path name of the current input file

- `__LINE__`: expand to the current input line number

- `__func__`: 获取函数名，GCC还支持 `__FUNCTION__` 以及会带上参数打印的 `__PRETTY_FUNCTION__`。

  defined as if by: `static const char __func__[] = "function-name";`

- `#<YOUR_MACRO>`: stringize the macro tokens

  ```c++
  #define MY_MACRO(...) func(#__VA_ARGS__)
  MY_MACRO(a); // func("a");
  ```

- variadic macros: declared to accept a variable number of arguments, for example:

  ```c++
  #define eprintf(...) fprintf (stderr, __VA_ARGS__)
  ```

  - `__VA_ARGS__`: replace all the tokens in its argument list including any commas
  - `##__VA_ARGS__`: **(non-portable GCC syntax)** if  variable macros matched 0 args, help to delete the previous comma.
  - `__VA_OPT__`: **(available only in GNU C, GNU C++ and C++20)** optionally expand tokens in a variadic macro if number of args greater than 0.
  
  ```c++
  #define MY_MACRO_1(first,...) func(first,__VA_ARGS__)
  #define MY_MACRO_2(first,...) func(first,##__VA_ARGS__)
  #define MY_MACRO_3(first,...) func(first __VA_OPT__(,) __VA_ARGS__)
  MY_MACRO_1(a); // `func(a, );  
  MY_MACRO_2(a); // `func(a);
  MY_MACRO_3(a); // `func(a);
  ```
  
- xxx




### Recursive

Macro is not able to call itself recursively. But the following code managed to do it:

```C
// max macro recursive depth 3*3*3=27
#define EXPAND(...)             EXPAND3(EXPAND3(EXPAND3(__VA_ARGS__)))
#define EXPAND3(...)            EXPAND2(EXPAND2(EXPAND2(__VA_ARGS__)))
#define EXPAND2(...)            EXPAND1(EXPAND1(EXPAND1(__VA_ARGS__)))
#define EXPAND1(...)            __VA_ARGS__

// enable recursive macro (work with dummy helper)
#define PAREN                   ()

#define FUNC_MACRO_DUMMY() FUNC_MACRO
#define FUNC_MACRO(FIRST, ...)  DO_STH_WITH(FIRST) \
    __VA_OPT__(, FUNC_MACRO_DUMMY PAREN (__VA_ARGS__))

#define EXAMPLE(...)  EXPAND(FUNC_MACRO(__VA_ARGS__))
```



### using `do {...} while(0);` in macros

使用 `do {...} while(0);` 将目标语句语句包裹，保护宏定义替换后不会受大括号，分号，if，循环等影响
