https://godbolt.org/

# C++ compile stages

1. Preprocessing: convert *.cpp/c.* files to *.i* files

   Process`#` instructions: replace `#define`, delete comment, insert `#include` files, etc. A translation unit is the output of the preprocessor.

2. Compilation: convert *.i* files to *.s* files

   Translate source code to target assembly code.

3. Assembly: convert *.s* files to *.o* files

   Translate assenbly code to machine instructions (data seg + code seg) and symbol tables. 

   > Only check **declaration**! In symbol table contents: 
   >
   > - symbol name
   > - can/connot referenced by external (static is invisible to other object files)
   > - defined/not defined (not defined variable/func will be checked in linking stage)
   > - location - data/code segment

4. Linking: no specified output name, but default to *a.out*

   Linker takes one or more object files, and combines them to an executable file, a library file, or another object file. Linker checks if, for every object file, every undefined symbol occurs **exactly once** in other object files. 

   > - Stabic linked library: statically included in executable files (linux -- *.a* file; windows -- *.lib* file)
   > - Dynamic linked library: run-time dynamic link(linux -- *.so* file; windows -- *.dll* file)

   > 函数的实现尽量不要放在头文件，不过事实上现在的C++编译器完全可以自动处理类实现写入.h文件的情况，即使实现的成员函数前面是virtual之类不能inline的类型也不会有问题，最多只是降低编译速度而已。



# Application Binary Interface(ABI)

ABI 是编译器和链接器遵守的一组规则，以让编译后的程序可以正常工作。ABI里包含很多方面的内容，比较重要的有：

- name decoration：编译后的函数名。C++需要名字修饰的原因是因为C++允许函数重载，同名函数在C语言中会冲突，必须想办法让他们在编译器层面区分开来（namespace、类名等在名字修饰中都会有体现）名字修饰发生在编译阶段，既目标文件中的符号已经是修饰之后的。名字修饰没有规范，直接调用修饰后的函数名很容易导致二进制不兼容。
- Object Representation，一般说不稳定的 API，都是在说 内存布局不稳定···
- Function Calling Sequence（函数调用约定）：函数调用约定里涉及到寄存器怎么使用，参数如何传递（堆栈还是寄存器），谁负责清理堆栈（调用者/被调用者），参数入栈的顺序（从右向左？），栈帧的布局等。
- Data Representation：定义了系统基本类型的数据宽度，如 bool 类型定义以及 long 的字节数 （ILP） 等

C语言也受ABI困扰，不同 libc 就有不同 ABI，导致需要重新编译。 C++里 ABI 问题比较明显主要是因为语言设计就不考虑abi问题，而模板导致这个问题被放大，因为模板的设计就天然和abi冲突。



 # Compiler Optimization

> 代码性能优化是否应该依赖编译器？因为无法保证在其他编译器下就能得出跟当前类似的优化效果，也有观点认为应该手动修改代码实现来保证优化
>
> 依赖编译器优化的前提是开发人员了解编译器的优化机制或者说开发人员知道写怎样的代码能达到编译器优化的标准，但是，如果部门入职的新人接手了这块工作，在开发过程中，发现老代码都是直接返回(也就是作者口中的编译器优化)，然后新人也直接返回(即使编译器不会进行优化，例如前面的返回全局变量等)，就会得到相反的结果

 ### (N)RVO:

Return Value Optimization (RVO, 返回值优化) 是默认开启的编译器优化选项，可避免函数返回时产生临时对象，从而消除部分拷贝或移动操作，提高代码性能。可以通过编译选项 `-fno-elide-constructors` 关闭 (g++)。 优化后的函数会在调用点直接传入要构造的对象，也就是相当于直接构造出最终的对象，消除了两次拷贝或者移动操作以及两次析构操作。如：

```c++
// 原函数，调用方法： Obj o = fun();
Obj fun() {
 Obj obj(1);
 return obj;
}
// 优化后等价的调用方法：Obj o; fun(o);
void fun(Obj &_obj) {
 Obj obj(1);
 _obj.Obj::Obj(obj); 
 return;
}
```

更彻底的：

```c++
Obj fun() { // 原函数
 return Obj(1);
}
void fun(Obj &_obj) { // 优化后
 _obj.Obj::Obj(1);
}
```

Effective Modern C++ 中将 RVO 优化的条件概括为两点： **局部对象**的类型和返回类型相同+局部对象就是返回值

NRVO（Named RVO），与 RVO 的区别其实就是函数中返回的对象已经具名了，如：

```c++
Obj fun() {
	Obj o;
	return o;
}
```

> **(N)RVO 受限的场景：** 
>
> - 作为返回值时，（Linux平台下）不能使用std::move()，否则会无法应用(N)RVO 优化。Adding `move` prevents the move being elided, because `return std::move(foo);` is *not* eligible for NRVO. 会多产生一次拷贝和一次析构。
> - 但是如果不是 local 对象而是外部传入的类型，还是应当在必要时使用 move

### Copy elision optimization

In C++17, copy elision was made mandatory in some situations

```c++
Foo f = Foo(); // copy ctor not called
```

???



# Keywords

### inline (keyword)

The `inline` keyword suggests the compiler to substitute the code within the function definition for every instance of a function call. 

Using `inline` can improve performance because elimination of the function call overhead. The compiler can also optimize functions expanded inline in ways that aren't available to normal functions.

Any functions can be inline. Member functions defined within class definition are implicitly inline functions.

Virtual functions can also be inlined, but only when compiler knows the "exact" class of the object. (e.g. non-dynamic resolved, local object, static/global object, etc.)

编译器内部会有一个阈值，在编译一个函数的时候，会计算每条语句所贡献的值（这个值编译器内部有一个映射算法），加完看是否超过，超过就不做inline，不超过就inline。当然这中间还有一些check看看是否能inline之类的。不同的优化选项这个值会不一样，来tradeoff 性能和空间。



# GCC & G++

 *glibc* memory allocator is a C library released by GNU, containing functions like`malloc` and `free`. It talks to the OS kernal, request and release the virtual memory for the processes in a wise way. (e.g. request a large virtual memory from the OS, and allocate them to processes eventually)



# Clang





### C and C++ Libraries

> GCC vs Clang

##### standard C library

*libc* is the earlist standard C library in Linux, and is taken place by *glibc* eventually. 

*glibc* is the GNU's version of standard C library, it's the most popular one and is pre-installed in most Linux systems. *glibc* 实现了 Linux 系统中最底层的 API 库，主要是对系统调用的封装，比如 fopen。同时也提供了一些通用的数据类型和操作，如 string，malloc，signal 等。管理内存分配的函数如 malloc 和 free。 

除此之外还有一些小众 C 库，如用于嵌入式环境的 eglibc，轻量级的 glib 等，它们在 Linux 中不是预装的。

##### standard C++ libraries

*libstdc++* is the standard C++ library of gcc and *libc++* is the standard C++ library of clang.

clang 重写的 libc++ 库比 libstdc++ 更充实，但两者不能兼容。在大多数以 GCC 主导的操作系统中，clang 默认使用的是 gcc 的标准 C++ 库来编译程序，也就是 libstdc++，如果需要使用 libc++，要额外在编译参数中设置。这可能是处于兼容性的考虑。

libstdc++ 是和 gcc 绑定安装的，但 glibc 和 gcc 却没有绑定安装，这是因为 glibc 过于底层，在不同硬件上不能通用，所以绑定安装可能会导致危险的问题，而 libstdc++ 就显得没那么底层了。

##### runtime library

运行时库包含标准库。标准库是程序语言要求的基础功能集合，通常它是独立于不同硬件的，因为语言需要保证一定的可移植性，所以编程语言定出来的库规范，一定是能具有通用性的；但运行时库是需要保障软件在硬件上正常运行的，依据不同的硬件，运行时库的实现可能不同。运行时库对标准库做了扩展，支持软件能够在系统上正常运行。所以，查看标准库规范应该到 C 标准委员会或 C++ 标准委员会的网站上查询，而查看运行时库，需要到对应编译器的手册或运行时库自己的手册中查询。

> 原子操作通常都是硬件强依赖的，所以通常都需要编译器的运行时库来提供支持。

GCC 的运行时库是 libgcc_s，clang 的运行时库是 runtime-rt。如上一节提到的，clang 在大多数 GCC 主导的操作系统中默认使用 GCC 的标准库，同时它也默认使用 GCC 的运行时库。如果需要切换使用 clang 的标准库，那么要额外指定使用 clang 的 runtime-rt 才可以。这需要在编译时给定一些配置参数。当然，你也可以选择把两个版本的运行时库都链接到程序中，但这样通常是冗余和浪费的。

GCC 的运行时库相比 clang 的 runtime-rt，会缺少一些 LLVM 依赖的接口实现。

##### builtin functions

builtin函数是一种在编译器内部实现的，使用它并不需要包含#include<>头文件的支持，在编译的时候，直接变为指令块的替换操作，根据函数的所描述的功能匹配机器文件（MD）中的函数 对应的指令模板，如果找到合适的指令模板就会输出其中的汇编指令，否则就会调用标准库函数。如果编译器内部实现内建函数，那么在从汇编代码生成ELF可执行文件时，就不需要库函数的链接过程。这些内建函数一般是基于不同硬件平台采用专门的硬件指令实现的，因此性能较高。

> GCC compare and swap built-in functions are not supported by clang, for example



# Undefined Behavior (UB)

##### unsequenced modification and access

> C11: 6.5 Expressions:
>
> If a **side effect** on a scalar object is unsequenced relative to either a different side effect on the same scalar object or a value computation using the value of the same scalar object, the behavior is undefined. If there are multiple allowable orderings of the subexpressions of an expression, the behavior is undefined if such an unsequenced side effect occurs in any of the orderings.
>
> https://blog.csdn.net/qwe_lingkun/article/details/47311417
>
> https://stackoverflow.com/questions/32017561/unsequenced-modification-and-access-to-pointer

Example: `a[i]=a[++i]` , `printf("%d,%d/n", ++i, i++);`

- 不确定点1: 不确定编译器按什么顺序处理表达式，即使大部分编译器从右向左处理（RR parser？）
- 不确定点2: C语言中的++，可能是全部自增结束后依次入栈，也可能是依次先自增入栈再处理下个表达式



# Linux Practice

See [Linux Command Line](../Framework & Platform & Tools/Linux Command Line.md) chapter.