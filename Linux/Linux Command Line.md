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

- `arch`: architecture of processor (x86/arm)

- `free`: 显示系统整体内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。`-g`: in GB

- `vmstat`: 监视虚拟内存使用

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

- `htop`: 是 `top` 命令的增强版， 一启动就显示所有信息，使用颜色来区分不同的资源使用类型，并提供更丰富的快捷键，交互，批量操作，以及自定义排序筛选 (F1 for help)

  - CPU: Blue - low priority threads ｜ Green - normal priority threads | Red - Kernel threads

  - Memory: Green - used memory | Blue - buffers | Yellow - cache

- `ps -ef`: check process, 显示进程父，子，COMMAND

  > `-ef` vs `aux`: 如果关心是进程的资源使用，使用 `ps aux`；如果关心的是进程的父子关系或完整的启动命令，使用 `ps -ef`。
  >
  > - `a`: 显示所有用户的进程，包括其他用户的进程。
  > - `u`: 以用户友好的格式显示进程信息，包括用户、CPU 使用率、内存使用率、启动时间、进程状态等。
  > - `x`: 显示没有控制终端的进程（通常是后台进程）。
  > - `-e`: 显示所有进程（类似于 `ps aux` 中的 `a`）。
  > - `-f`: 以全格式显示进程信息，包括进程的父进程 ID (PPID) 和启动命令的完整路径。

- `ps aux`: 显示进程资源占用

  - STAT: 进程状态：(`man ps` 可查)
  
  - CPU使用率，为整个进程运行周期内平均值
  - VSZ: Virtual Memory Size, 虚拟内存的大小，包含了未被加载到实际内存中的空间
  - RSS: Resident Set Size, 真正被加载到物理内存中的页的大小（包含共享库的内存，如果把系统中所有进程的RSS相加会比总内存大）
  - PSS: Proportional Set Size, 将共享库的内存按使用的进程个数平均分成多份，ps 命令看不到
  - USS: Unique Set Size, 全部被该进程独占的内存大小，揭示了进程终止时实际被返还给系统的内存，是针对某个进程开始有可疑内存泄露的情况，进行检测的最佳数字。
  
- `sar`: 查看历史系统活动报告，包括 cpu 使用率，进程，内存，页面交换，串口使用等。

- `pstack`: 跟踪进程栈， 定位程序挂起/卡死位置

- `strace`: 监测系统调用数量和延迟

- `lsof`: 查看系统中打开的文件，无参数打印系统中所有 open file， `-p` 按 pid 筛选，`-u` 按用户筛选，`-i` 筛选所有网络连接，`-i tcp` 筛选所有 tcp 链接，`-i :port` 列出网络端口占用

- `ipcs`: 查看系统中信号量、共享内存、队列等 IPC 资源占用

- `iostat`: 查看 cpu，网卡，磁盘，tty 等设备的活动情况

- `kill`:

  - `-9`: 发送不可捕获和不可忽略的 `SIGKILL` 信号，直接导致进程立即终止，没有机会进行清理操作。

  - `-15`: 默认为此模式，发送请求进程终止的信号 `SIGTERM` ，进程可以捕获这个信号并执行一些清理操作然后正常退出。`SIGTERM` 给进程提供了一个机会来保存数据、释放资源等。 

- lspci: 可列出每个pci/pcie总线上的设备，通过grep过滤后可得到：

  - 网卡：`lspci | grep -i net`

  - 显卡：`lspci | grep -i vga`

  - Nvidia GPU：`lspci | grep -i nvidia` or `nvidia-smi`

- Network: see Linux-network chapter

- system service: `service --status-all`



# C/C++ related

> refer to C++/third-party-lib & tools/command-line-tools.md



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

- linux version: `uname -r`
- print linux system information: `lsb_release -a` or `cat /etc/issue` or `cat /etc/euleros-latest`
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
  - 挂载网络文件系统 (NFS): 

    - 首先确保挂载路径已在 */etc/exports* 注册
    - 修改 */etc/exports* 后记得重启 NFS 服务 `exportfs -a`
    - 即可正常挂载 NFS, e.g. `mount -t nfs server_ip:/shared_directory /mnt/nfs_shared`
