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

- awk: 一个强大的文本分析工具

  - 获取第一个字段：`ask '{print $1}'`



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

  > 删除软链接时后面不要带上斜杠 /，会导致被链接文件夹中文件被删除

- 硬链接，全称叫作硬链接文件，英文名称是 hard link。`ln`默认创建的是硬链接，这类文件比较特殊，会拥有自己的 inode 节点和名称，其 inode 会指向文件内容所在的数据块。与此同时，该文件内容所在的数据块的引用计数会加 1。当此数据块的引用计数大于等于 2 时，则表示有多个文件同时指向了这一数据块。一个文件修改，多个文件都会生效。当删除其中某个文件时，对另一个文件不会有影响，仅仅是数据块的引用计数减 1。当引用计数为 0 时，则系统才会清除此数据块。

### Users and groups

> When creating a new user, the default behavior of the `useradd` command is to create a group with the same name as the username, and same GID as UID. The `-g` (`--gid`) option allows you to create a user with a specific initial login group. You can specify either the group name or the GID number. The group name or GID must already exist.

```sh
# Check users and groups
cat /etc/passwd
cat /etc/group
groups $your_user_name
groups $your_group_name
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

### Files & users & permission

- 修改所有者: `chown -R 1003:1003 /path/to/dir`
- 变更访问权限: `chmod -R 777 /path/to/dir`



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

- Network

  - iperf: a tool for active measurements of the maximum achievable bandwidth on IP networks
  - ifconfig: displays information about all network interfaces currently in operation
  - IP/port usage: `netstat -tulpn | grep LISTEN`
  - specific ports: `lsof -i:10002`
  - Check ssh service: `ps -ef | grep ssh`

- system service: `service --status-all`



# C/C++ related

##### gcc/g++

```shell
g++ -o ./helloworld ./hello.cpp
./helloworld
g++ ./hello.cpp
./a.out # default executable
```

- `-S`: generate assembly code *filename.s*
- `-g`: enable debug

##### clang

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

##### LDD

LDD (List Dynamic Dependencies) is a unix utility that prints the shared libraries required by each program or shared library specified on the command line. 检查依赖库是否齐全

LDD Search:

1. **Standard library search paths:** The dynamic linker maintains a list of standard search paths where it looks for shared objects. These paths are typically defined in the linker configuration files (`/etc/ld.so.conf` or `/etc/ld.so.conf.d/*.conf`) and include directories like `/lib`, `/usr/lib`, and `/usr/lib64`.
2. **Environment variable `LD_LIBRARY_PATH`:** If set, the `LD_LIBRARY_PATH` environment variable specifies a list of additional search paths for shared objects. These paths are searched before the standard paths.
3. **RPATH**: Some executables or shared objects may contain an embedded RPATH (runtime path) that specifies additional search paths for their dependencies. This information is extracted from the ELF header of the file. Using the `readelf -d <so_file>` command to display the ELF header of a shared object, which includes the RPATH.
4. **Explicit path provided to `ldd`:** When you run `ldd` with an explicit path to a file, it directly searches for dependencies using that path. This can be useful for debugging or troubleshooting issues with shared libraries.



# other commands

- print linux system information: `lsb_release -a` or `cat /etc/issue` 

- apt withou root: `apt download <pkg>` & `dpkg -x <pkg>.deb <dir>`

- 定时任务: Crontab
  - vim编辑创建定时任务：`crontab -e`

  - 从文件导入：`crontab filename`

  - 查看单个用户任务：`crontab -l -u usrname`

  - 查看所有用户的任务：`cat /etc/passwd | cut -f 1 -d : |xargs -I {} crontab -l -u {}`




# Environment Variable

> https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html

`export XX=/xx/xx` affacts environment variables; `XX=/xx/xx` does not.

- `PATH`：系统搜索可执行程序的路径
- `LDFLAGS`：gcc等编译器搜索库文件的路径
- `CFLAGS`：gcc等编译器搜索头文件的路
- `CFLAGS`, `CXXFLAGS`, `CPPFLAGS`: the default rules in make pass`CPPFLAGS` to C PreProcessor (means it's always applied), while `CFLAGS` is only passed to C compiler, and `CXXFLAGS` is only passed to C++ compiler
- `LIBS` or `LDLIBS`: 告诉链接器要链接哪些库文件，如LIBS = -lpthread -liconv

PATH变量的分隔符是`:`，其他的是空格

