# Components

STL 是一个标准，对接口进行规范，其实现可以有不同版本。目前流行的 STL实现如 SGI STL 版本被 GCC 采用，此外还有如 Visual C++ 采用的 P.J. Plauger 版本等。STL 的目标就是要把数据和算法分开，分别对其进行设计，之后通过 iterator 把这二者再粘接到一起。

- 容器（containers）
- 算法（algorithms）
- 迭代器（iterators）
- 仿函数（functors）协助算法完成各种操作
- 配接器（adapters）用来套接适配仿函数
- 空间配置器（allocator）给容器分配存储空间

上述各部分收录在 STL/core 目录



# STL implementations

- HP STL: developed by Alexandar Stepanov and Meng Lee in Hewlett-Packard (HP) Lab, Palo Alto, 1990. C++ STL的第一个实现。后续其他版本的 STL 深受其影响，大都以其为蓝本。
- SGI STL: Developed by Silicon Graphics International (SGI) in the mid-1990s. Main designer, STL's father - Alexandar Stepanov. A most influential early implementation of the STL.
- GNU's STL(libstdc++): default C++ STL implementation for GCC compiler (influenced by SGI STL)
- LLVM's STL (libc++): default C++ STL for Clang compiler.
- Microsoft STL: The default C++ Standard Library for Microsoft's Visual C++ compiler (MSVC).
