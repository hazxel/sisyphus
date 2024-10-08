# POSIX

POSIX（Portable Operating System Interface）是一个标准化的操作系统**接口规范**，由 IEEE（Institute of Electrical and Electronics Engineers）制定。它旨在提高操作系统之间的兼容性，使得程序能够在不同的操作系统上移植和运行，而无需修改代码。POSIX 主要定义操作系统的 **API**、工具和接口而非具体实现，使得在遵守该标准的系统上编写的程序可以更容易地迁移和运行。

> Ubuntu 和 CentOS：作为 Linux 系统的代表，广泛支持 POSIX 标准及其扩展。
> macOS：良好地支持 POSIX 标准，大多数基础功能和扩展都被实现，但其文件系统和系统调用有自己的实现和扩展。
> Windows：无原生 POSIX 标准支持，但可以通过 WSL、Cygwin、MinGW 等工具或子系统提供 POSIX 兼容的环境。



# IPC

Inter-process communication is a mechanism provided by the OS for communications between several processes.

> Transmission Modes:
>
> - simplex: Only one of the two parties on a link can transmit, the other can only receive.
> - half-duplex: each party can both transmit and receive, but not at the same time.
> - full-duplex: both parties can transmit and receive simultaneously.



# Fork

### POSIX API：

- `fork`: create a new process. If success, returns 0 to the child process and returns the PID of the child process to the parent process.  Otherwise return -1 to the parent process, no child process is created, and the global variable errno is set to indicate the error.
- `_exit(0)`: terminate the calling process (`0`表示正常退出)
- `wait(nullptr)`: for parent process to wait a child process' termination(either one)
- `waitpid`: wait for a specific child process' termination



# pthread？？？



# Signal

The only asyncronous communication in IPC.  It's complex and used to notify the process that an event has occurred. It's like the **software-level** interrupt. A process receiving a signal is like a processor receiving an interrupt.

信号是操作系统定义的很短的消息类型，通常用一个数字表示信号。信号需要通过内核转发到相应进程，而后内核会为进程执行约定的操作。进程可以将信号设置为阻塞或忽略，也可以自定义信号处理函数来捕获信号，没有定义时会采用下列之一对应的缺省行为：

- Terminate：kill the process
- Dump：kill the process and dump core to a file if possible (useful when debugging)
- Ignore: nothing happens
- Stop & Continue: change process status to running/stop

典型的信号有：

- SIGKILL：强迫进程终止，不可被忽略，捕获，或阻塞
- SIGSTOP：强迫进程挂起，不可被忽略，捕获，或阻塞

POSIX 还规定了某些信号必须发送给进程中的所有线程，如 SIGKILL, SIGCHLD, SIGQUIT 等



# Semaphore

a **counter** that can be used to control access to shared resources by multiple threads.

P and V are atomic operations, P(sv) will decrease sv by 1 and suspend if sv=0, while V(sv) will increase sv by 1 and invoke the suspended process.

- Advantages: can synchronize processes
- Disadvantage: just a counter, not very expressive

### Anonymous semaphore

Anonymous semaphore: used within a process to sync threads

Its lifecycle is controlled by the process. allocated within the process's memory, won't affect IPC namespaces.

- `sem_init`: initialize an unnamed semaphore
- `sem_destroy`: destroy an unnamed semaphore
- Delete: anonymous semaphore doesn't need explicit delete 

### Named semaphore: can be shared between processes

Named semaphore is given a name ，命名信号量在文件系统的 IPC 虚拟命名空间中注册，这使得不同进程可以通过名字访问同一个信号量。(类似于在文件系统中创建一个文件)

- `sem_open`: initialize and open a named semaphore, 一般选一个进程来open，确保其他进程仅打开

  - `name`: the identifier of the semaphore, usually start with `/`, e.g. `"/my_semaphore"`
  - `oflag`: specifies flags that control the operation of the call.  frequently used flags:
    - `O_CREAT`: created if doesn't exist 
    - `O_EXCL`: If both`O_CREAT` and `O_EXCL` are specified in oflag, then an `EEXIST` error is returned if a semaphore with the given name already exists.
  - `mode`: permissions settings for the semaphore, must supply if oflag includes `O_CREAT`
  - `value`: initial value for the semaphore, must supply if oflag includes `O_CREAT`

  returns the address of the new semaphore  `sem_t *` on success, otherwise `SEM_FAILED`

- `sem_close`:  关闭当前进程对该命名信号量的描述符。其他进程仍可通过其信号量描述符访问该信号量

- `sem_unlink`: 删除命名信号量，实质上的删除在所有打开该信号量的进程关闭它后触发。首次删除后，后续删除不会改变系统状态。

- `sem_wait`: decrements (locks) the semaphore.

- `sem_post`: increments (unlocks) the semaphore



# share memory

A common memory created in RAM, and can be accessed by multiple processes.

- Advantages: fastest, bidirectional, large amount of information, can be used by multiple processes
- Disadvantages: Require concurrency control (e.g. semaphore, mutex)

### POSIX API

- `shm_open`: creates a new, or opens an existing shared memory object 实际的内存尚未分配

  - `name`: the identifier of the shared memory object, usually start with `/`, e.g. `"/my_shm"`
  - `oflag`: specifies flags that control the operation of the call.  frequently used flags:
    - `O_CREAT` & `O_EXCL`: similar to semaphore
    - `O_RDONLY` & `O_RDWR` Open the object for *read-only* or *read-write* access.
    - `O_TRUNC`: If the shared memory object already exists, truncate it to zero bytes.
  - `mode`: permissions settings for the shared memory

  returns **a file descriptor** referring to the shared memory object with `FD_CLOEXEC` flag set, and is normally used in subsequent calls to `ftruncate` (if newly created object) and `mmap`.  After a call to `mmap`, the file descriptor may be closed.

- `ftruncate`: truncate a file to a specified length

- `mmap` & `munmap`: map or unmap files or devices into memory

- `close`: close a file descriptor

- `shm_unlink`: removes a shared memory object name, and, once all processes have unmapped the object, deallocates and destroys the contents of the associated memory region.



# pipe

half-duplex

A pipe is an important mechanism in Unix-based systems that allows us to communicate data from one process to another without storing anything on the disk. The following two kinds of pipes are very similar except for creating and deleting.

- Named Pipe (FIFO):

  A named pipe can last as long as the system is up. Usually a named pipe appears as a file, and generally processes attach to it for IPC.

  - Advantages: allows communication between unrelated processes
  - Disadvantages: Long-term storage in the system, improper use is prone to errors. Limited buffer

- Anonymous pipe (pipe): `|`

  Typically a parent program opens anonymous pipes, and creates a new process that inherits the other ends of the pipes, or creates several new processes and arranges them in a pipeline. When no process is holding a reference to a pipe, it will be closed automatically.

  - Advantages: simple and convenient
  - Disadvantages: Can only be created between its processes and their related processes. Limited buffer

### API？？



# Message Queue

a linked list of messages, stored in the kernel and identified by the message queue identifier

A data stream similar to a socket, but which usually preserves message boundaries. Typically implemented by the operating system, they allow multiple processes to read and write to the message queue without being directly connected to each other.

- Advantages: Easy to implement. Allow communication between any process. Send and Receive through system call, and no need to consider synchronization issues.
- Disadvantages: Copying information requires additional CPU time, which is not suitable for situations with large amounts of information or frequent operations



# Socket

can be used for process communication between different computers

Socket 是对 TCP/IP 协议族的一种封装，是应用层与TCP/IP协议族通信的中间软件抽象层。从设计模式的角度看来，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

- Advantages:
  1. The transmission data is byte level, the transmission data can be customized, the data volume is small and the efficiency is high
  2. Short data transmission time and high performance
  3. Suitable for real-time information exchange between client and server
  4. Can be encrypted, strong data security
- Disadvantages: The transmitted data needs to be parsed and converted into application-level data.



# Others

- `sleep`: 影响整个进程，所有线程全部挂起。现代 c++ 中建议使用 ``std::this_thread::sleep_for` 以确保更好的跨平台兼容性和代码的可维护性。



# syscall

系统调用在 Linux 内核中实现，他们的定义可以在 Linux 内核源代码中找到。系统调用通常通过 `::syscall` 接口进行访问，而不是通过标准的 C 库或用户空间库头文件直接定义。可以在 `/usr/include/bits/syscall.h` 找到 syscall 的定义。

`::systemcall` 的入参是用户进程传递的系统调用号，返回值是一个整数的错误码（和 `::open`  这些 C 标准库的封装例程不同，系统调用不设置 `errno` 变量）

### mmap & brk - 详见 Memory 章节

### msync

把共享内存映射的脏页刷新到磁盘。

### IPC syscall

- semget
- msgget
- shmget

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

