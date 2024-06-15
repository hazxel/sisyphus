# Command line flow control

- Linear (`;`) Run commands one by one

- And (`&&`) If previous return 0 (success), then run next command

- Or (`||`) If previous doesn't succeed, run next.

- Pipe (`|`) Pass the STDOUT to STDIN of next command. only handle std output, no std error. 

- use the output of another command: `$()` or \`\`

- xargs

  converts input from standard input into arguments to a command.

  ```shell
  $ find /path -type f -print | xargs rm
  $ find . -name "*.cpp" | grep "is.cpp" # different from pure pipe!!
  $ find . -name "*.cpp" | xargs grep "include <vector>"
  $ cat args.txt | xargs -I {} ./script.sh {}
  ```



# Text Commands

##### Streams and redirection: Stdin(0), Stdout(1), and Stderr(2)

The corresponding numerical identifier values of stdin, stdout, and stderr are 0, 1, and 2.

- `>`& `1>`: redirect stdout to ...
- `2>`: redirect stderr to ...
- `&1` & `&2`: escape symbol, otherwise would redirect to a file named 1 or 2

##### commands

- cut: 从某一行信息中取出某部分我们想要的信息。分隔符，取第几段...

- grep: 分析文件中一行信息，如果其中有我们需要的信息，就将该行拿出来

  ```
  $ grep "string" name.file
  $ grep -E "regex" name.file
  $ grep -r "with line number" name.file
  ```

- cat: 读取和合并文件，并将其内容写入到标准输出或文件。

  ```
  cat /etc/issue				#写入标准输出
  cat < /etc/issue				#写入标准输出
  cat file.txt > file2.txt 	#创建并覆盖
  cat file.txt >> file2.txt 	#重定向
  ```

- echo: 用于字符串的输出，可显示字符串（可转义），环境变量等

- tail: prints data from the end of a specified file or pipe input. Provides an easy way to quickly see file updates in real time, as new data is usually added to the end of a file.

- wc: count total words, lines, characters. `-l` count total lines

- tr: replace string `tr '\n' ','` 换行变逗号

- awk: 一个强大的文本分析工具

  - 获取第一个字段：`ask '{print $1}'`
  
- sed: 也是强大的字符串处理工具

  - 替换：`sed 's/find-str/replace-to/g'`
  - 整行替换（正则）：`sed 's/^.*find-str.*$/replace-to/g'`
  - 有时要对正则转义：`sed 's/[0-9]\+/replace/g'`
  



# File Commands

- `diff`：比较文件或文件夹（需要添加`-r`参数）
- `wget`: 下载文件`wget <参数> <URL地址>`, `--no-check-certificate` if cert failed
- `file`: determine the type of a file and its data. The command doesn't take the file extension into account, and instead runs a series of tests to discover the type of file data.
- `scp`: copy from remote
  - `scp -r username@servername:/var/www/remote_dir/ /var/www/local_dir`
  - `scp username@servername:/path/filename /var/www/local_dir`
- `tar`: 压缩/解压 
  - *.tar* files: `tar xvf FileName.tar` &`tar cvf FileName.tar DirName`
  - *.tar.gz* files：`tar zxvf FileName.tar.gz` & `tar zcvf FileName.tar.gz DirName`
  - *.tar.xz* files: `tar xf FileName.tar.xz`
  - 去掉外层文件夹：`--strip-components=1`

### ln 链接

- 软链接，全称是软链接文件，英文叫作 symbolic link。需要提供`-s` 参数来创建，非常类似于 Windows 里的快捷方式，这个软链接文件（假设叫 VA）的内容，其实是另外一个文件（假设叫 B）的路径和名称，当打开 A 文件时，实际上系统会根据其内容找到并打开 B 文件。

  - 删除软链接时后面不要带上斜杠 /，会导致被链接文件夹中文件被删除
  - 软链接的源路径不能为另一个软链接
  - `ln -s /src/path /dst/path` 中，如果 `/dst/path` 是一个文件夹，就会在文件夹下创建一个和源文件同名的软链接，如果不是文件夹而是一个（不存在的）文件，就会以该名称创建软链接

- 硬链接，全称叫作硬链接文件，英文名称是 hard link。`ln`默认创建的是硬链接，这类文件比较特殊，会拥有自己的 inode 节点和名称，其 inode 会指向文件内容所在的数据块。与此同时，该文件内容所在的数据块的引用计数会加 1。当此数据块的引用计数大于等于 2 时，则表示有多个文件同时指向了这一数据块。一个文件修改，多个文件都会生效。当删除其中某个文件时，对另一个文件不会有影响，仅仅是数据块的引用计数减 1。当引用计数为 0 时，则系统才会清除此数据块。





# Users and groups

### Commands:

- 列举所有用户: `cat /etc/passwd`

- 列举所有用户组: `groups` / `cat /etc/group`

- 列举用户所属组: `groups $user_name`

- 列举组内用户: `groups $group_name`

- 列举当前用户uid gid 等信息: `id`

- 创建用户: `adduser $user_name`

  > `useradd` is native binary compiled with the system, while `adduser` is a perl script wrapper which uses `useradd` binary in back-end. `adduser` is more user friendly and interactive.
  >
  > When creating a new user, the default behavior of the `useradd` command is to create a group with the same name as the username, and same GID as UID. The `-g` (`--gid`) option allows you to create a user with a specific initial login group. You can specify either the group name or the GID number. The group name or GID must already exist.

- 修改/设置密码: `passwd $user_name`

- 切换用户switch user: `su $user_name` 不带用户名则切换为 root 

- 修改文件的 所有者/所有组: `chown -R 1003:1003 /path/to/dir`

- 修改文件的 所有者/用户组/其它用户 的 读/写/执行权限: `chmod -R 777 /path/to/dir`

### create user

```sh
#Create your own account
sudo adduser $your_user_name
sudo passwd $your_user_name
# authorize home dir
sudo chown $your_user_name:$your_user_name -R /home/$your_home_dir
# Add your account to docker group
sudo usermod -aG docker $your_user_name
sudo newgrp docker
sudo su $your_user_name
```

### groups

通常不使用 root 用户直接登录，而是用普通用户登录。需要 root 权限时，可以 su 登录成为 root 用户。但只要知道了 root 的密码，任何人都可以登录为 root 用户，有安全隐患。

wheel 组类似于一个管理员的组，普通用户加入 wheel 后就会成为管理员组内的用户，一般会配置只有 wheel组的用户才可以用 su 登录为 root，以增强系统安全性：

- 修改 /etc/pam.d/su 文件，找到“#auth required /lib/security/$ISA/pam_wheel.so use_uid ”这一行，将行首的“#”去掉。
- 修改 /etc/login.defs 文件，在最后一行增加“SU_WHEEL_ONLY yes”语句。
- 用“usermod -G wheel 用户名”将一个用户添加到wheel组中。



# Vim

- 跳转文件头：`gg`
- 跳转文件尾：`shift + g`
- 剪切一行：`dd`
- 粘贴在下一行：`p`

Vim 可按层级浏览文件夹，压缩文件，甚至jar包（本质上是个压缩文件）



# 资源监控

- `nproc`: number of processor

- `free -g`: 显示系统整体内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。`-g`: in GB

- `fdisk -l`: 查看新插入的磁盘是否被识别

- `df -h`: 查看磁盘占用

- `du -h`: 查看当前目录下的子目录和文件大小

- `mount /dev/sdb1 /home/disk`: 临时挂载磁盘，reboot会失效，开机自动挂载需写入 fstab:

  ```shell
  PARTITION=/dev/sdb1 	# 挂载分区
  MOUNT_PATH=/home/disk # 挂载目录
  if [[ -n "$fdisk -l | grep ${PARTITION}" ]] && [[ -d "${MOUNT_PATH}" ]] && [[ "$(cat /etc/fstab | grep "^${PARTITION}" | wc -l)" -eq 0]]; then
  	echo "${PARTITION} ${MOUNT_PATH}   exts    defaults     0  0" >> /etc/fstab
  	cat /etc/fstab
  fi
  ```

- `top`: 分析进程性能，包括 pid, user, priority, virtual memory, shared memory, state(S/R/I/...), CPU, Mem, time, command。按 1 可查看瞬时CPU占用

- `ps -ef`: check process, 显示进程父，子，COMMAND

- `ps aux`: 显示进程资源占用

  - CPU使用率，为整个进程运行周期内平均值
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

- iostat

- lspci: 可列出每个pci/pcie总线上的设备，通过grep过滤后可得到：

  - 网卡：`lspci | grep -i net`

  - 显卡：`lspci | grep -i vga`

  - Nvidia GPU：`lspci | grep -i nvidia` or `nvidia-smi`

- Network: see Linux-network chapter

- system service: `service --status-all`



# C/C++ related

### Compiler options

- include directories `-I` or `isystem`: specifying the directories in which header files are located. Ususally stored in environment variable `CPPFLAGS`
- preprocessor flags `-D`: specifying preprocessor definitions. Ususally stored in environment variable `CPPFLAGS`
- other compiler options: control behavior of the compiler. Ususally stored in environment variable `CFLAGS` or `CXXFLAGS`
  - `-std=c++11` for specifying the C++ language standard
  - `-E`: generate preprocessed code
  - `-S`: generate assembly code *filename.s*
  - `-g`: enable debug
  - `-I`: provide include path
  - `-shared`: compile to shared lib
  - `-pie ` & `-no-pie`: whether the executable is position independent. 不加载到内存固定位置，提升程序安全性
  - `-Wall`： enabling all warnings
  - `-Werror`: all warnings treated as errors 有助于确保代码质量，因为它迫使开发人员处理所有的警告
  - `-Wl,aaa,bbb,ccc`: a comma-separated list of tokens that will be passed to linker as space-separated list as `ld aaa bbb ccc`

### GNU Linker (`ld`) options

linker `ld` is usuallly called automatically by compiler, but can be also called manually. `-l` and `-L` given to the compiler will be passed to linker.

- `-rpath`: add a directory to the **runtime** library search path, will be passed to the runtime linker to find *.so* file at runtime 

- library directory flags `-L`: add a path to the **linking-time** search path that *ld* will search for archive libraries and control scripts. Ususally stored in environment variable `LDFLAGS`

- libraries flags `-l`: The names of libraries to be linked. Ususally stored in environment variable `LIBS` or `LDLIBS`

  > **Order matters here!!**
  >
  > The linker searches from left to right, and notes unresolved symbols as it goes. If a library resolves the symbol, it takes the object files of that library to resolve the symbol. Dependencies of static libraries against each other work the same - the library that needs symbols **must be first**, then the library that resolves the symbol.
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





# Environment Variable

`export XX=/xx/xx` affacts environment variables; `XX=/xx/xx` does not. How to add paths to 

### Important Environment Variables

> more refer to: https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html

- `PATH`：系统搜索可执行程序的路径
- `LDFLAGS`：gcc等编译器搜索库文件的路径
- `CFLAGS`：gcc等编译器搜索头文件的路
- `CFLAGS`, `CXXFLAGS`, `CPPFLAGS`: the default rules in make pass`CPPFLAGS` to C PreProcessor (means it's always applied), while `CFLAGS` is only passed to C compiler, and `CXXFLAGS` is only passed to C++ compiler
- `LIBS` or `LDLIBS`: 告诉链接器要链接哪些库文件，如LIBS = -lpthread -liconv

> PATH变量的分隔符是`:`，`export PATH="$PATH:<PATH 1>:...:<PATH N>"`其他环境变量一般是空格

### Priority of loading environment variable coifigure files:

```shell
/etc/profile
/etc/bashrc & /etc/zshrc
/etc/paths 
~/.bash_profile # macOS
~/.bash_login 
~/.profile 
~/.bashrc & ~/.zshrc # linux
```

When you need to add a environment variable, better not to add directly to **/etc/paths** , but to create a new file in directory **/etc/paths.d** , and include your desired path, for example: `/opt/homebrew/bin`. Similarly, modification to **/etc/profile** , **/etc/bashrc** and **/etc/zshrc** are not recommended.

If you are executing your files like `sh 1.sh` or `./1.sh`, you are executing it in a sub-shell. If you want the changes on environment variable to take effect in current shell, run: `. 1.sh` or `source 1.sh`





# package management

### apt get

- apt withou root: `apt download <pkg>` & `dpkg -x <pkg>.deb <dir>`

- 有时一个 package 需要公钥验证才能下载：

  - 下载公钥：`wget https://xx.xx.xx.gpg`

  - 修改 apt source 配置：新增 `/etc/apt/sources.list.d/xxx.list` 文件，内容大致为：

    ```
    deb [signed-by=/etc/path/to/key] https://download.src/stable/distro/arch/xxx
    ```

- xxx



# other commands

- print linux system information: `lsb_release -a` or `cat /etc/issue` or `cat/etc/euleros-latest`
- 定时任务: Crontab
  - vim编辑创建定时任务：`crontab -e`

  - 从文件导入：`crontab filename`

  - 查看单个用户任务：`crontab -l -u usrname`

  - 查看所有用户的任务：`cat /etc/passwd | cut -f 1 -d : |xargs -I {} crontab -l -u {}`
- `pushd`/`popd`: 替代 `cd`，用一个栈存储目录 （`cd`  改变的是栈顶）
- To figure out the default shell: `echo $SHELL`


  - To figure out all available shells: `cat /etc/shells`

  - Switch between shells:`chsh -s /bin/bash`,`chsh -s /bin/zsh`


  - Alias: defined in your shell configuration file, and act as a shortcut to reference a frequently used command, for example: `alias v="vim"`
  - 无法 reboot 时，重启服务器：`echo "b" > /proc/sysrq-trigger`
