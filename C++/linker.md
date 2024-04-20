# Executable and Linkable Format (ELF)

> vs Bin / Hex ???

ELF is the executable and linkable format used in Linux, Android, BSD, Solaris, etc. These are all ELFs:

- executable file
- object file (`.o`, relocatable file)
- static library (`.a`, a collection of relocatable files)
- shared library (`.so`, shared object file)
- core dump: ??? consists of the recorded state of the working memory of a computer program at a specific time, generally when the program has crashed or otherwise terminated abnormally.

### File layout:

ELF 文件的组成部分中，只有ELF 文件头的位置固定在开头，其它内容的位置不定。

- ELF header: 位于文件开头，包含有整个文件的结构信息
- program header table: 在运行过程中是必须的，在链接过程中是可选的，作用是告诉系统如何创建进程的镜像，包含每个 program segment and relevant info.
- section header table: 在链接过程中是必须的，在运行过程中是可选的，包含每个section header and relevant info .
- Data referred to by entries in the program header table or section header table

> 注意区分段 (Segment) 和节 (Section)。汇编程序中 `.text`，`.bss`，`.data` 这些指示都是指 Section。
>
> Section 和 Segment 从不同角度看待同一个ELF文件，这个在ELF中被称为不同的视图 (View)。Section （给linker）提供链接时所需的信息（Linking View），Segment （给OS）提供运行时所需的信息（Execution View）。

### Section

包含有链接时需要的指令数据、符号数据、重定位数据等，如：符号表 `.symtab`、字符串表 `.strtab`、动态符号表 `.dynsym`、动态字符串表 `.dynstr`、代码 section `.text`、data `.data`、BSS  `.bss`

### Segment

ELF把具有相同读写/执行权限的 section 合并成一个segment，一个 segment 可能包括0个或多个 sections。合并后的 segment 会作为一个整体映射到虚拟内存空间和实际物理空间，并且一个Segment对应一个VMA。ELF中主要的权限组合：（R 可读，W 可写，E 可执行）

- R Segment: `.rodata`
- R+E Segment: `.text`, `.plt` ...
- R+W Segment: `.data`, `.bss`, `.got`, `.got.plt`,  ...

segment 一般没有具体的名字，只有编号。此外，只有 type 为 `LOAD` 的段是运行时需要的。



# Linker and Loader

`ld` is a general-purpose linker used in the Unix and Unix-like operating systems to combine objects and archive files, relocates their data and ties up symbol references. Usually the last step in compiling a program is to run `ld`.

- Static library is linked into the executable at compile time, resulting in a static binary.

- For shared library, there are two options:

  - dynamic linking (commonly used): Specify the shared library the program uses when compiling the executable (otherwise won't compile). When program starts, OS will open these libraries.

  - dynamic loading: the program itself open the shared library and invoke the desired function. Such programs are usually linked with *libdl*, which provides the ability to open a shared library.


后续再研究：运行时谁来装载到内存？？？怎么装？？？



# Linkage of symbols

A object, reference, function, type, template, namespace, or value, may have *linkage*.

- **No linkage**: The name can be referred to only from the scope it is in.
- **External linkage**: The name can be referred to from the scopes in the other translation units.
- **Internal linkage**: The name can be referred to from all scopes in the current translation unit.
- **Module linkage** (C++20): The name can be referred to only from the scopes in the same module unit or in the other translation units of the same named module.

最终，

- 全局符号以自己的符号地址寻址

- 静态符号都是以所在的段基地址为base进行寻址的，这样做的好处就是全局符号表中只需要记录全局符号和段符号就行了，大量的仅内部可见符号就不用记录了。

  - 静态变量以其所在的 .data 段基地址作为基地址寻址

  - 静态指针以其所在的 .data.rel.local 段或 .data.rel 段基地址作为基址寻址
  - 静态函数在编译的时候就完成地址绑定，链接期不需要处理



# Static Linking

源码先要编译成目标文件.o，再链接成可执行文件或共享对象。Linking view 下，目标文件由 *段* 构成，段内对函数或变量的引用分为如下两种类型处理：

- 全局函数或变量（文件内部或外部定义的全局符号）：编译时作为外部符号处理，在重定位表中有相应的描述项，由LD统一进行重定位。

- 静态函数或变量（文件内部定义的静态符号）：静态变量也会被当作外部符号处理，因为它们最终会被 LD 合并成到可执行文件的 bss segment 或 data segment，相对地址会变化；静态函数定义在本编译单元，也不受 LD 过程合并同类项影响，在编译的时候就可完成地址绑定，所以为内部符号。

### 链接时重定位

对于外部符号的引用，编译期无法得知其地址，所以编译期会放置一个占位符。如果在目标文件 `.o` 或是静态库`.a` 中定义，那么在链接阶段可以确定其地址，按相对引用来修正先前放置的占位符，这就是重定位。



# Dynamic Linking

### 运行时重定位

和链接时重定位不同，如果目标符号定义在动态库内，虽然链接器可以知晓其定义在哪个动态库，但仍然无法确定其地址，故重定位需要延后至运行时。然而运行时我们无法再修改代码段，只能修改数据段。

因此，内存中会有一个可读写段 GOT，在运行时存放真正的符号地址，GOT 段本身的地址在链接时确定；为了感知 GOT 中的真正地址，链接阶段发现符号定义在动态库时，链接器还会生成 PLT，其本质上是一些 stub 代码，通过确定地址访问 GOT，获取真正符号地址并完成跳转。链接时，链接器会使用 PLT 中 stub 代码的地址取代编译阶段就已生成好的 CALL 指令中的占位符。

### GOT (Global Offset Table，全局偏移表)

GOT 的每一项都是一个外部符号的实际地址，用于定位全局变量和函数。GOT表可以发生变化。 `.got` is for relocations regarding global 'variables' while `.got.plt` is a auxiliary section to act together with `.plt` when resolving procedures absolute addresses.

其中，`.got.plt` 表中前三项有特别的用处：

- GOT[0]: 自身模块的 dynamic 段地址
- GOT[1]: 本模块的 link_map 地址
- GOT[2]: 系统模块中的 `_dl_runtime_resolve` 函数地址
- 从第4项开始，就是正常的外部函数地址 `func@got.plt`

### PLT(Procedure Link Table) : 过程链接表

PLT 属于代码段，表中每一项都是一小段代码，主要内容为 JMP 跳转以及 PUSH 堆栈。其指向 GOT 表的逻辑在编译时已完全确定，在进程加载和运行时不会改变。

##### for lazy binding: `.plt`

`.plt` 代码段的起始段被预留，依次存放以下三条指令：

- `push X`:  压栈 `_dl_runtime_resolve` 的第一个参数，是一个 `link_map` 结构体的指针
- `jmpq *0x2fe4(%rip)`:  使用到 GOT 中预留的第二项 GOT[2] 的偏移，获取存放的 `_dl_runtime_resolve` 函数的地址并跳转
- `nopl`: 不做任何操作，用于指令对齐

后续的依次为正常表项 `func@plt` ，每项依次存放以下三条指令：

- `jmp *0x1234(%rip)`: 使用到 GOT 中 `func@got.plt` 的偏移，获取存放的地址并跳转（这条是正常寻址调用函数的 stub 指令）
- `push X`: 压栈该 `func` 对应的编号。 `_dl_runtime_resolve` 一共两个参数，这是第二个参数（这条指令和下一条指令为延迟绑定时首次调用函数的 stub 指令）
- `jmp <.plt>`: 跳转 `.plt` 的起始预留代码段

##### for non-lazy binding: `.plt.got`

？？？大概就是直接在加载程序时将真实地址写入 GOT 表，后续再补充

### 延迟绑定 (Lazy Binding)

所谓延迟绑定，就是当函数第一次被调用的时候才进行符号查找和重定位。如果函数从来没有用到过就不进行绑定，可大大加快程序的启动速度，特别有利于一些引用了大量函数的程序。

- 第一次调用前，GOT 的表项 `func@got.plt` 并没有被填充到函数的真正地址，而是 PLT 中的另一段 stub 代码，一般紧贴在`func@plt` 下方 +6 个字节。

  - 这段代码会调用 `_dl_runtime_resolve`，而这个函数会：

    - 解析 `func` 的真实地址并写入 `.got.plt` 表

    - 跳转执行该函数

- 这样下次调用该函数就可根据真实地址直接跳转了



# 符号

符号中会包含 namespace

编译目标文件时，符号表可用来检查重复定义。链接可执行文件时，链接器会去符号表查找引用的符号是否存在。编译共享库时，符号检查会推迟到运行时，（undefined symbol 标记为 U 但不报错）



# 安全相关：？？？后续再看

因为 got 表是可写的，容易被劫持攻击，因此衍生出一系列安全措施：

PIC

DSO：Dynamic shared object

**RELRO（RELocation Read Only）**

在Linux中有两种RELRO模式："Partial RELRO" 和 "Full RELRO"。Linux中Partical RELRO默认开启，

KASLR：Kernel Address Space Layout Randomization

segment randomized offset?
