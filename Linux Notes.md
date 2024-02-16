Unix哲学KISS：keep it simple, stupid。在Linux系统里，一切看上去十分复杂的逻辑功能，都用简单到不可思议的方式实现，甚至有些时候看上去很愚蠢。但仔细推敲，人们将会赞叹Linux的精巧设计，或许这就是大智若愚。

# command line instructions

### Command line flow control

- Linear (`;`) Run commands one by one
- And (`&&`) If previous return 0 (success), then run next command
- Or (`||`) If previous doesn't succeed, run next.
- Pipe (`|`) Pass the STDOUT to STDIN of next command. only handle std output, no std error. 
- use the output of another command: `$()` or \`\`

### Streams

##### Stdin(0), Stdout(1), and Stderr(2)

The corresponding numerical identifier values of stdin, stdout, and stderr are 0, 1, and 2.

##### redirection

- `>`& `1>`: redirect stdout to ...
- `2>`: redirect stderr to ...
- `&1` & `&2`: escape symbol, otherwise would redirect to a file named 1 or 2

### Basic commands

##### xargs

converts input from standard input into arguments to a command.

```Sh
$ find /path -type f -print | xargs rm
$ find . -name "*.cpp" | grep "is.cpp" # different from pure pipe!!
$ find . -name "*.cpp" | xargs grep "include <vector>" 
```

##### cut

从某一行信息中取出某部分我们想要的信息。分隔符，取第几段...

##### grep

分析文件中一行信息，如果其中有我们需要的信息，就将该行拿出来

```sh
$ grep "被查找的字符串/正则" 文件名
```

##### cat

读取和合并文件，并将其内容写入到标准输出或文件。

```sh
cat /etc/issue				#写入标准输出
cat < /etc/issue				#same
cat file.txt > file2.txt 	#创建并覆盖
cat file.txt >> file2.txt 	#重定向
```

##### echo

用于字符串的输出，可显示字符串（可转义），环境变量等

##### print linux system information: 

`lsb_release -a` or `cat /etc/issue` for example:

> Distributor ID:	Ubuntu
>
> Description:	Ubuntu 18.10
>
> Release:	18.10
>
> Codename:	cosmic

##### ln

- 软链接，全称是软链接文件，英文叫作 symbolic link。需要提供`-s` 参数来创建，非常类似于 Windows 里的快捷方式，这个软链接文件（假设叫 VA）的内容，其实是另外一个文件（假设叫 B）的路径和名称，当打开 A 文件时，实际上系统会根据其内容找到并打开 B 文件。

  > 删除软链接时后面不要带上斜杠 /，会导致被链接文件夹中文件被删除

- 硬链接，全称叫作硬链接文件，英文名称是 hard link。`ln`默认创建的是硬链接，这类文件比较特殊，会拥有自己的 inode 节点和名称，其 inode 会指向文件内容所在的数据块。与此同时，该文件内容所在的数据块的引用计数会加 1。当此数据块的引用计数大于等于 2 时，则表示有多个文件同时指向了这一数据块。一个文件修改，多个文件都会生效。当删除其中某个文件时，对另一个文件不会有影响，仅仅是数据块的引用计数减 1。当引用计数为 0 时，则系统才会清除此数据块。

##### wget

usage: `wget <参数> <URL地址>`

##### tar

- *.tar* files: `tar xvf FileName.tar` &`tar cvf FileName.tar DirName`
- *.tar.gz* files：`tar zxvf FileName.tar.gz` & `tar zcvf FileName.tar.gz DirName`
- *.tar.xz* files: `tar xf FileName.tar.xz`

##### file

The Linux `file` command helps determine the type of a file and its data. The command doesn't take the file extension into account, and instead runs a series of tests to discover the type of file data.

##### tail

The Linux `tail` command is a command-line utility that prints data from the end of a specified file or files to standard output. The utility provides an easy way to quickly see file updates in real time, as new data is usually added to the end of a file.

##### apt

- withou root: `apt download <pkg>` & `dpkg -x <pkg>.deb <dir>`

##### scp

`scp -r username@servername:/var/www/remote_dir/ /var/www/local_dir`

`scp username@servername:/path/filename /var/www/local_dir`

##### nm

nm 命令显示关于指定 File 中符号的信息，文件可以是对象文件、可执行文件或对象文件库。所谓符号，通常指定义出的函数，全局变量等等。有用的options:

- A 在每个符号信息的前面打印所在对象文件名称

- C 输出demangle过了的符号名称 (overloading of C++)

- D 打印动态符号(for .so/.dylib)

  > By default, `nm` reads the `.symtab` section in ELF objects, which is optional in non-relocatable objects. With the [`-D`/`--dynamic` option](https://sourceware.org/binutils/docs/binutils/nm.html#index-dynamic-symbols), you can instruct `nm` to read the dynamic symbol table (which are the symbols actually used at run time). You may also want to use `--with-symbol-versions` because glibc uses symbol versioning extensively.

- l 使用对象文件中的调试信息打印出所在源文件及行号

- n 按照地址/符号值来排序

- u 打印出那些未定义的符号

```shell
nm -uCA *.o | grep foo
nm -A /usr/lib/* 2>/dev/null | grep "T memset"
```

##### gcc/g++

```shell
g++ -o ./helloworld ./hello.cpp
./helloworld
g++ ./hello.cpp
./a.out # default executable
```

- `-S`: generate assembly code *filename.s*

##### clang

##### Network

- iperf: a tool for active measurements of the maximum achievable bandwidth on IP networks
- ifconfig: displays information about all network interfaces currently in operation

##### Hardware Resource

- top: pid, user, priority, virtual memory, shared memory, state(S/R/I/...), CPU, Mem, time, command

- `ps -ef`: 显示进程父，子，COMMAND

- `ps aux`: 显示进程资源占用
  
  - VSS: Virtual Memory Size, 虚拟内存的大小，包含了未被加载到实际内存中的空间
  - RSS: Resident Set Size, 真正被加载到物理内存中的页的大小（包含共享库的内存，如果把系统中所有进程的RSS相加会比总内存大）
  - PSS: Proportional Set Size, 将共享库的内存按使用的进程个数平均分成多份，ps 命令看不到
  - USS: Unique Set Size, 全部被该进程独占的内存大小，揭示了进程终止时实际被返还给系统的内存，是针对某个进程开始有可疑内存泄露的情况，进行检测的最佳数字。
  
  > memory info all read from `/proc/meminfo`, 
  >
  > - buffer: data that has yet to be "written" to disk. 
  > - cache: data has been "read" from the disk and stored for later use. 在操作系统中指 page cache，页高速缓存，内核实现的磁盘缓存，把对磁盘的访问变为对物理内存的访问
  > - swap: 当内存不足的时候，把一部分硬盘空间虚拟成内存使用。内存的容量有限，只需要当文件的某部分内容真正被访问到时再调入内存（demand paging）
  >
  > 活跃和非活跃的内存页分为两种：
  >
  > - file-backed pages（文件背景页）
  > - anonymous pages（匿名页）
  >
  > 进程的代码段、映射的文件都是**file-backed**，而进程的堆、栈不与文件对应，属于匿名页。RSS包括匿名页
  >
  > file-backed pages在内存不足的时候可以直接写回对应的硬盘文件里，称为page-out，不需要用到交换区(swap)；而anonymous pages在内存不足时就只能写到硬盘上的交换区(swap)里，称为swap-out。
  
- free: 显示系统整体内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。

- iostat





# Environment Variable

> https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html

- `PATH`：系统搜索可执行程序的路径
- `LDFLAGS`：gcc等编译器搜索库文件的路径
- `CFLAGS`：gcc等编译器搜索头文件的路
- `CFLAGS`, `CXXFLAGS`, `CPPFLAGS`: the default rules in make pass`CPPFLAGS` to C PreProcessor (means it's always applied), while `CFLAGS` is only passed to C compiler, and `CXXFLAGS` is only passed to C++ compiler
- `LIBS` or `LDLIBS`: 告诉链接器要链接哪些库文件，如LIBS = -lpthread -liconv

PATH变量的分隔符是`:`，其他的是空格




# Process

### create process：

- fork: copy all the resources of father

  > Copy-on-write (COW): 为了降低开销，fork最初并不会真的产生两个不同的拷贝，因为在那个时候，大量的数据其实完全是一样的。写时复制是在推迟真正的数据拷贝。若后来确实发生了写入，那意味着parent和child的数据不一致了，于是产生复制动作，每个进程拿到属于自己的那一份，这样就可以降低系统调用的开销。所以有了写时复制后呢，vfork其实现意义就不大了。 

- vfork: parent and child share the same address space

- clone: a call with parameter (clone_flags). Can select partial resource of parent to share with child

### process status

- R：running，教科书一般将正在CPU上执行的进程定义为RUNNING状态、可执行但是尚未被调度执行的进程定义为READY状态，这两种状态在linux下统一为 TASK_RUNNING状态。
- Z：僵尸状态，进程在退出的过程中，处于TASK_DEAD状态。 在这个退出过程中，进程占有的所有资源将被回收，除了task_struct结构（以及少数资源）以外。于是进程就只剩下task_struct这么个空壳，故称为僵尸。
- S (TASK_INTERRUPTIBLE)，可中断的睡眠状态。
- D (TASK_UNINTERRUPTIBLE)，不可中断的睡眠状态。
- T (TASK_STOPPED or TASK_TRACED)，暂停状态或跟踪状态
- X：退出状态，进程即将被销毁

