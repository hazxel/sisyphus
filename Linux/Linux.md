# Unix

Unix 哲学 KISS：keep it simple, stupid。一切看上去十分复杂的逻辑功能，都用简单到不可思议的方式实现，甚至有些时候看上去很愚蠢。但仔细推敲，人们将会赞叹其精巧的设计，或许这就是大智若愚。

除 Linux 外，其他 Unix 系统还有 Solaris, FreeBSD, OpenBSD, NetBSD, MacOS 等。各种 Linux 版本大都遵从一些通用标准，如 IEEE's POSIX (Portable Operating Systems based on Unix),  X/Open's CAE (Common Application Environment, 公共应用环境)等，但这些标准并不包括内核的实现。



# File



# Service

- SSH: default to 22 port, *sshd* stand for SSH daemon and is the server-side process for SSH, configuration in `/etc/ssh/sshd_config`



# Users and Groups

### user

Linux user information is in `/etc/passwd`

### group

linux group information is in `/etc/group`

### permission

In Linux, file permissions are represented numerically with a three-digit octal value. Each digit corresponds to a different category of users:

1. **Owner**: The permissions for the file's owner (user).
2. **Group**: The permissions for the group that the owner belongs to.
3. **Others**: The permissions for all other users.

Each digit in the octal representation is an octal (base-8) number, which translates to a 3-bit binary number. This binary number represents the permissions as follows:

- **Read (r)**: The permission to read the file. (1 in the binary representation)
- **Write (w)**: The permission to modify the file. (2 in the binary representation)
- **Execute (x)**: The permission to execute the file. (4 in the binary representation)



# syscall

系统调用在 Linux 内核中实现，他们的定义可以在 Linux 内核源代码中找到。系统调用通常通过 `::syscall` 接口进行访问，而不是通过标准的 C 库或用户空间库头文件直接定义。可以在 `/usr/include/bits/syscall.h` 找到 syscall 的定义。

### io_uring

3 related syscall: 

- `io_uring_setup`: 初始化并分配内存
  - `u32 entries`: the minimum size of SQ and CQ
  - `struct io_uring_params *p`: contains options passed to kernel, other fields filld by kernel
  - return an `int` fd for subsequent operations on the io_uring instance
- `io_uring_enter`: 提交请求到内核执行
- `io_uring_register`:  注册文件描述符或缓冲区，使其可在 I/O 操作中被引用，避免重复传递，并且由于已提前 mmap，也能减少延迟

encapsulation & helper provided by liburing: 

- `io_uring_queue_init`: calls `io_uring_setup` and xxx ???
- `io_uring_submit`: calls `io_uring_enter`
- `io_uring_prep_*`: prepare SQE，包含所有异步 I/O 操作需要的信息。
- `io_uring_submit`: 一般仅更新 SQ，某些情况下 call `io_uring_enter` 来唤醒内核线程 `io_sq_thread` 
- `io_uring_wait_cqe`: block and loop to wait for a new CQE (no sys call)
- `io_uring_cqe_seen`: update CQ head, kernel will handle processed entries (no syscall)
- `io_uring_register_files`, `io_uring_register_eventfd`: calls `io_uring_register`
- `io_uring_queue_exit`: `munmap` 解除提交队列和完成队列的共享内存内存映射，`close` 关闭 ring 的文件描述符，（io_uring 实例本身似乎是内核负责释放？？？）
