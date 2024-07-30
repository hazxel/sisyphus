# String Literal

String literal is stored in data segment after compilation. 

- can initialize `const char*`, also is the real type of the string literal.
- can initialize `char[]`. This will result in a **copy** from data segment to stack.
- cannot be implicitly converted to non-const `char*` or `wchar_t*` since C++11.



# char array

array都是在栈上储存，超出作用域会自动销毁



# String

### Split

STL string don't have a split function. Current work around is:

```c++
istringstream ss{str};
string field;
while (getline(ss, field, ';')) { // or ss >> field with splitter set to ' '
    cout << field << '\n';
}
```

??? SSO?https://zhuanlan.zhihu.com/p/547694685

### StringStream

- `str()`: 
  - 无参数时，get string content from stringstream
  - 有参数时，set string content of stringstream，可用于清空：`ss.str("");`

### String 的一堆大麻烦

- 为啥很多人不用string 而用char数组？
- u8string相关？
- ？？？



# String View(C++17)

https://blog.csdn.net/MMTS_yang/article/details/130773313

`std::string_view` 对象引用一个外部的字符序列，本质上是指针+长度。典型场景：

- 你可能已经分配或者映射了字符序列或者字符串的数据，并且想在**不分配更多内存的情况下使用这些数据**。典型的例子是内存映射文件或者处理长文本的子串（处理大文件时，先全部读到内存，后续使用 view 来查看，避免创建新的 string）
- 你可能想提升**接收字符串为参数并以只读方式使用它们**的函数/操作的性能，且这些函数/操作不需要结尾有空字符。

**字符串视图远比字符串引用或者智能指针更危险**！它们的行为更近似于原生字符指针。在使用时必须保证引用的字符串序列是有效的，否则稍不注意就会产生运行时错误。

```c++
std::string_view bad_get() {
  char ar[] = "bad example";
	return ar;
} // runtime error when cout << bad_get();
```

string view 本身是小值，应当 pass-by-value 传递，不要使用引用

### vs `const std::string&`

`std::string&` 调用 `substr()` 时，一定会通过拷贝创建一个新 `std::string`。

把函数参数声明为`std::string_view`，与声明为`std::string`比较起来，**可能**会减少一次分配堆内存的调用

- `const std::string&` 接收字符串字面量时，会创建一个**临时**的`string`，这将**会在堆上分配一次内存**，除非传递的是短字符串，且使用了短字符串优化(SSO)
- 通过使用字符串视图，将不会分配内存，因为字符串视图只 *指向* 字符串字面量。





# format & vformat (C++20)

### `std::format`

- 格式字符串 format 中的占位符 `{}` 会被后续参数所替换。要求 format 字符串是编译时常量（`consteval` 实现，因为要检查 `{}` 和后续参数数量是否一致）
- `std::format` 底层由 `std::vformat` 实现

### `std::vformat`

`std::vformat` 是一个与 `std::format` 类似的函数，但它接受一个 `std::format_args` 对象，而不是一组参数。这使得它非常适合于动态生成参数列表的场景，比如在函数中接收可变数量的参数。

```c++
std::string fmt = "Name: {}, Age: {}";
auto args = std::make_format_args("Alice", 30);
std::string result = std::vformat(fmt, args);
```



# Static String Best Practice

- char pointer: `const char* str_ptr = "abc";`
- char array: `char str_array = "abc";`
- string: `std::string str = "abc";`
- string view: `std::string_view = "abc";`





# STL I/O

### Stream

A **stream** is a flow of data into or out of a program.

- `<ios>`: Input-Output base classes like `ios_base` and `ios`.

- `<istream>`: contains input stream `istream`; also `iostream` which inherits `istream` and `ostream`.

- `<ostream>`: contains output stream `ostream`

- `<iostream>`: `cin` is an instance of an `istream`; `cout`, `cerr` and `clog` are instances of `ostream`

 ```c++
extern istream cin; // so as cout/cerr/clog
 ```

- `<streambuf>`: Stream buffers represent input or output devices and provide a low level interface for unformatted I/O to that device.

- `<fstream>`: contains `ifstream`, `ofstream`, `fstream` and `filebuf`, which inherit correspondingly `isteam`, `ostream`, `iostream` and `streambuf`. They are used to read/write data from/to files.

- `<sstream>`: contains `istringstream`, `ostringstream`, `stringstream` and `stringbuf`, which inherit `isteam`, `ostream`, `iostream` and `streambuf`, and they provide advanced string operations.

Useful Functions: 

- `getline`: defined in `<string>`, reads a line of characters (separated by `\n` or other specified delimiter) from an input stream and places them into a string. An example that to split a strings with specified splitter:

  ```c++
  std::string str = "a,b,c";
  std::stringstream ss(str);
  while(getline(ss, str, ','))
  	std::cout << str << "\n";
  ```

- `operator>>`: usually read a word (separated by whitespace, `\n`, etc) from stream.

- `flush`: ensures that all data that has been written to that stream is output, including clearing any that may have been buffered.

### alpha

- `boolaplha` & `noboolaplha`: passed to any derived type of `std::basic_ostream` or receive any derived type of `std::basic_istream`. 

### `\n` vs `std::endl`

`std::endl` flushes the output buffer, and `\n` doesn't. If you don't want the buffer flushed frequently, use `\n`.



# Files

### C++ file operations

Stream based, just like `cin` and `cout`.

- ofstream
- ifstream
- fstream

### C file operations

- fopen & fclose
- fputc & fgetc
- fputs & fgets