# C/C++ related

### Compiler options

- include directories `-I` or `isystem`: specifying the directories in which header files are located. Ususally stored in environment variable `CPPFLAGS`
- preprocessor flags `-D`: specifying preprocessor definitions. Ususally stored in environment variable `CPPFLAGS`
- other compiler options: control behavior of the compiler. Ususally stored in environment variable `CFLAGS` or `CXXFLAGS`
  - `-std=c++11`: specifying the C++ language standard, default value varies, better explicitly specify
  - `-E`: generate preprocessed code
  - `-S`: generate assembly code *filename.s*
  - `-g`: enable debug
  - `-I`: provide include path
  - `-shared`: compile to shared lib
  - `-O`: optimization level, usually defaults to `-O0`
  - `-pie ` & `-no-pie`: whether the executable is position independent. 不加载到内存固定位置，提升程序安全性
  - `-Wall`： enabling all warnings
  - `-Werror`: all warnings treated as errors 有助于确保代码质量，因为它迫使开发人员处理所有的警告
  - `-Wl,aaa,bbb,ccc`: a comma-separated list of tokens that will be passed to linker as space-separated list as `ld aaa bbb ccc`
  - `-fsanitize=address`: enable gcc address sanitizer (ASAN) to detect memory faults

### gdb

- `break` or `b`: 设置断点
- `continue` or `c`: 继续执行，直到下一个断点或结束
- `backtrace` or `bt`: 显示当前的函数调用栈
- `frame`: 输出当前的栈帧信息，包括函数名、源文件和当前代码的行号
- `frame <n>`: 切换到指定栈帧
- `list`: 查看执行的代码上下文
- `info`：
  - `info locals`: 查看该栈帧中定义的所有局部变量
  - `info args`: 查看当前栈帧中传递给函数的参数
- xxx

### GNU Linker (`ld`) options

linker `ld` is usuallly called automatically by compiler, but can be also called manually. `-l` and `-L` given to the compiler will be passed to linker.

- `-rpath`: add a directory to the **runtime** library search path, will be passed to the runtime linker to find *.so* file at runtime 

- library directory flags `-L`: add a path to the **linking-time** search path that *ld* will search for archive libraries and control scripts. Ususally stored in environment variable `LDFLAGS`

- libraries flags `-l`: The names of libraries to be linked. Ususally stored in environment variable `LIBS` or `LDLIBS`

  > **Order matters here!!**
  >
  > The linker searches from left to right, and notes unresolved symbols as it goes. If a library resolves the symbol, it takes the object files of that library to resolve the symbol. Dependencies of static libraries against each other work the same - the library that needs symbols **must be first**, then follows the library that resolves the symbol. cmake 中也是，被依赖的库需要写在后面
  >
  > If a static library depends on another library, but the other library again depends on the former library, there is a cycle. You can resolve this by enclosing the cyclically dependent libraries by `-(` and `-)`, such as `-( -la -lb -)` (need to escape the parens, such as `-\(` and `-\)`). The linker then searches those enclosed lib multiple times to ensure cycling dependencies are resolved. Alternatively, you can specify the libraries multiple times, so each is before one another: `-la -lb -la`.
  >
  > https://stackoverflow.com/questions/45135/why-does-the-order-in-which-libraries-are-linked-sometimes-cause-errors-in-gcc
  >
  > > this apply also to CMake's `target_link_library` because it result in `-lxxx` after all.

- `-pie ` & `-no-pie`: whether the executable is position independent. 不加载到内存固定位置，提升程序安全性

- `-z`: followed by keywords:

  - `relro` & `norelro`: whether memory segement is read-only after relocation 
  - `now`: mark for runtime linker, that the shared library or executable should resolve all symbols when program started


### nm

nm 命令显示关于指定 File 中符号的信息，文件可以是对象文件、可执行文件或对象文件库。所谓符号，通常指定义出的函数，全局变量等等。有用的options:

- `-A` 在每个符号信息的前面打印所在对象文件名称

- `-C` 输出demangle过了的符号名称 (overloading of C++)

- `-D` 打印动态符号(for .so/.dylib) **否则有时候会 no symbols**

  > By default, `nm` reads the `.symtab` section in ELF objects, which is optional in non-relocatable objects. With the [`-D`/`--dynamic` option](https://sourceware.org/binutils/docs/binutils/nm.html#index-dynamic-symbols), you can instruct `nm` to read the dynamic symbol table (which are the symbols actually used at run time). You may also want to use `--with-symbol-versions` because glibc uses symbol versioning extensively.

- l 使用对象文件中的调试信息打印出所在源文件及行号

- n 按照地址/符号值来排序

- u 打印出那些未定义的符号

```shell
nm -uCA *.o | grep foo
nm -A /usr/lib/* 2>/dev/null | grep "T memset"
```

### LDD

LDD (List Dynamic Dependencies) is a unix utility that prints the shared libraries required by each program or shared library specified on the command line. 检查依赖库是否齐全

LDD Search:

1. **Standard library search paths:** The dynamic linker maintains a list of standard search paths where it looks for shared objects. These paths are typically defined in the linker configuration files (`/etc/ld.so.conf` or `/etc/ld.so.conf.d/*.conf`) and include directories like `/lib`, `/usr/lib`, and `/usr/lib64`.
2. **Environment variable `LD_LIBRARY_PATH`:** If set, the `LD_LIBRARY_PATH` environment variable specifies a list of additional search paths for shared objects. These paths are searched before the standard paths.
3. **RPATH**: Some executables or shared objects may contain an embedded RPATH (runtime path) that specifies additional search paths for their dependencies. This information is extracted from the ELF header of the file. Using the `readelf -d <so_file>` command to display the ELF header of a shared object, which includes the RPATH.
4. **Explicit path provided to `ldd`:** When you run `ldd` with an explicit path to a file, it directly searches for dependencies using that path. This can be useful for debugging or troubleshooting issues with shared libraries.

查看库版本也可以直接运行 `./xxx.so`，会输出一些版本信息

### objdump

Different from *ld*, simply dumping what the object itself lists as libraries containing unresolved symbols. 可用于分析目标文件（object file）和可执行文件（executable file），可以显示二进制文件的汇编代码、符号表、段信息等，是理解程序底层实现、调试和逆向工程的有力助手。

### readelf

和 objdump 类似，但信息更具体且不依赖 bfd 库。

### ccache

ccache 是一个编译器缓存，可以将编译的结果缓存到本机目录下，如 `$HOME/.ccache`。这样之后再次编译将变得很快。

- 直接在命令行使用：`ccache gcc xx.c`

- 可使用编译器的名字为 `ccache` 创建软链接，此时调用 `gcc xx.c` 时，虽然实际调用的是 `ccache` 而非 `gcc`，但 `ccache` 会根据调用命令去 `PATH` 寻找名为 `gcc` 的编译器 (第二个参数不必再传编译器了)

  ```sh
  cp ccache /usr/local/bin/
  ln -s ccache /usr/local/bin/gcc
  ln -s ccache /usr/local/bin/g++
  ```

- 在 Makefile 或者 CMake 中，可以使用 `export CC="ccache gcc"` 还有 `export CXX="ccache g++"`

- CMake 中最近也直接支持直接声明为 launcher 以达成伪装成编译器的目的：`CMAKE_<LANG>_COMPILER_LAUNCHER` ，`CMAKE_<LANG>_LINKER_LAUNCHER`

