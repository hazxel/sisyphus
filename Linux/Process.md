# Process

###  process descriptor 进程描述符

Linux 将进程和线程视为同一种实体，统称为 task 任务。因此进程描述符对应的数据结构 `task_struct` 事实上存放了进程和线程的管理信息，是 Linux 下进程控制块 (PCB) 的具体实现。其中的字段：

- `pid`: 进程的 pid 或线程对应的轻量级进程的 pid，是内核感知的 pid
- `tgid`: 所属线程组的 id，即线程组中的主线程的 `pid`。这个字段是`getpid` 系统调用返回的值，也即用户感知的 pid (ps 命令打印的 pid)。
- `pgid`: 标识 process group 进程组，即一个或多个进程组成的集合
- `sid`: 标识 session 会话，即一个或多个进程组的集合
- `state`: 进程状态，主要有：
  - R：running，教科书一般将正在CPU上执行的进程定义为RUNNING状态、可执行但是尚未被调度执行的进程定义为READY状态，这两种状态在linux下统一为 TASK_RUNNING状态。
  - Z：僵尸状态，进程在退出的过程中，处于TASK_DEAD状态。 在这个退出过程中，进程占有的所有资源将被回收，除了task_struct结构（以及少数资源）以外。于是进程就只剩下task_struct这么个空壳，故称为僵尸。
  - S (TASK_INTERRUPTIBLE)，可中断的睡眠状态。
  - D (TASK_UNINTERRUPTIBLE)，不可中断的睡眠状态。
  - T (TASK_STOPPED or TASK_TRACED)，暂停状态或跟踪状态
  - X：退出状态，进程即将被销毁
- `thread_info`: 指向线程信息结构体的指针，

单线程进程被看作只有一个线程的线程组，它的 `task_struct` 代表整个进程，也代表该进程唯一的线程。

内核需要同时处理很多线程和进程，因此进程描述符被分配在动态内存，通过双向循环链表相连，该链表的头是 idle 进程的描述符。特别的，内核将每个进程的 `thread_info` 与该进程在内核态下的堆栈存放在相邻的两个页框中 (内核控制路径只使用很少的栈，8KB已足够)，`thread_info` 处于该内存区底部，而栈从顶部向下增长。这样做的主要好处是可以方便的从标志栈顶地址的寄存器 esp 获得 `thread_info` 起始地址 (屏蔽掉低 13 位)。

### create process

传统的 Unix 操作系创建进程时总是复制父进程的所有资源，效率非常低。Linux 有三个相关的系统调用，不同程度上解决了该问题 (fork 的 cow，vfork共享内存，clone 传特定 flag 创建 LWP)。他们最终都会调用核内函数 `do_fork()`，但只有 `fork` 是 POSIX 规定的 API

- fork: copy all the resources of father

  > Copy-on-write (COW): 为了降低开销，fork最初并不会真的产生两个不同的拷贝，因为在那个时候，大量的数据其实完全是一样的。写时复制是在推迟真正的数据拷贝。若后来确实发生了写入，那意味着parent和child的数据不一致了，于是产生复制动作，每个进程拿到属于自己的那一份，这样就可以降低系统调用的开销。所以有了写时复制后呢，vfork其实现意义就不大了?

- vfork: parent and child share the same memory space; guarantees that child will execute first 父进程会在子进程执行完 `exec` 或 `exit` 之前被阻塞，以避免父子进程竞争修改相同的地址空间。

- clone: a call with parameter (clone_flags). Can select partial resource of parent to share with child 没有复制的数据通过指针让子进程共享

内核执行 `clone`,  `fork` , `vfork` 系统调用时，最终都会调用内核函数 `do_fork`，只不过传递的 flag 不同。glibc 在实现它们的 wrapper 时直接全部使用了 clone 系统调用，视他们为 clone 的特殊情况。

### process execution 进程执行

`execve` 是系统调用，负责执行新程序并替换当前进程的映像。它读取 ELF文件，把其中的代码和数据部分放到链接时指定的内存位置，然后从指定的开始地址执行代码：

1. 创建一个独立的虚拟地址空间
2. 读取可执行文件头，少部份 segment 直接加载进内存，其余 segment 建立虚拟空间与可执行文件的映射，后续通过缺页中断加载进内存。
3. 将CPU的指令寄存器设置成可执行文件的入口地址，启动运行

用户态 C 标准库的库函数，如 `execl`, `execle`或`execv`，封装了 `execve` 系统调用。



# thread 线程

### 线程模型

线程可以完全由用户实现，内核不感知，被称为用户级线程；也可以依赖内核的支持，被称为内核级线程。实际使用时，用户感知到的线程可能是用户自己管理的用户级线程，也可能是一层抽象，实际对应到内核级线程。根据用户线程和内核线程的对应关系，大致分为以下三种模型：

##### Many-to-One Model 多对一模型

将多个用户级线程（即同个进程内的所有线程）映射到一个内核级线程，完全依赖用户空间的线程库进行线程的建立，同步，销毁，切换，调度，内核无感知，因此开销小，效率高。但这种实现面对许多问题，因此较少被使用：

- 同一时间只能有一个线程被调度执行，并行性能不佳，无法利用多核处理器优势
- 一个线程执行阻塞的 IO 会导致所有线程阻塞，只能使用复杂的非阻塞技术保证整个进程可运行

##### One-to-One Model 一对一模型

每个用户级线程一对一映射一个内核级线程，提供了更好的并行性能，使线程可以在多个核心上并行，且一个线程阻塞时不会影响其他线程。但如果创建过多线程，会导致内核资源紧张，所以这种模型实现时一般需要限制系统内总线程的数量。

##### Many-to-Many Model 多对多模型

多对多模型下，多个用户线程对应多个内核级线程，用户线程数大于或等于内核的调度实体。在这种实现规避了上述两个模型的劣势。然而这种模型增加了线程实现的复杂性，可能出现的问题：

- 优先级反转
- 线程库和内核都可以进行线程调度，很难进行协调

Linux 内核的早期版本没有提供多线程的支持，用户进程的多线程管理就是采用多对一模型 (使用 pthread 库)。目前硬件性能的提升让开发者越来越不必担心内核线程的开销，因此大部分的操作系统比如 Linux 和 Windows 都采用一对一线程模型（但 Linux 内核中的调度单位只有进程，但有对线程的特殊支持；Windows 只调度线程，进程用于资源分配）。Solaris 曾经采用的 two-level model 两级线程模型就是一种多对多模型，不过其在 Solaris 9 版本后也转为采用一对一模型。

### 轻量级进程 (LWP)

LWP 不是 Linux 发明的，而是一个 general 的概念，指的是共享某些资源的一些进程。Linux 内核在 2.0 版本引入了轻量级进程，可以使用 clone 系统调用并传递 `CLONE_VM` flag 创建。`CLONE_VM` 意味着共享内存描述符和所有的页表，与父进程共享进程地址空间。除此之外 `CLONE_FS`、`CLONE_FILES`、`CLONE_SIGHAND` 等标志可以指定其他资源是否共享。

Linux 下 pthread 库（glibc 的一部分）提供的 LinuxThreads、NPTL 和 NGPT 都使用轻量级进程实现。

### LinuxThreads

LinuxThreads 是早期 Linux 上对 POSIX 标准线程库的实现，于 1996 年加入 glibc，NPTL 引入后被取代。

LinuxThreads 的实现基于一对一线程模型，将线程和轻量级进程直接关联，即每个用户感知的线程实体在内核角度都对应一个轻量级进程，而用户线程之间的管理在核外的用户使用的线程库中实现。其本质是通过 `clone` 系统调用，使用 `CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND` 参数创建线程。

LinuxThreads 使用一个 manager thread 来管理其他线程的创建，这会导致：

- 线程创建、销毁、同步时额外的上下文切换开销
- manager thread 只能在一个核上运行，在 SMP or NUMA 场景下线程同步开销大

LinuxThreads 未实现线程组，内核不感知线程和进程的差异，在信号/调度/IPC 等方面不符合 POSIX 标准：

- 一个进程创建的所有线程会有不同的 pid/uid/gid，系统内线程的总数量也会有限制 (4090 on IA32)
- 没有实现向进程中所有线程发送信号（内核以进程为单位分发异步信号）

### Native PosixThread Library (NPTL)

Linux 2.6 版本引入了 NPTL，一个真正的内核级线程库，提供了对 POSIX 线程标准（`pthread`）的原生支持，目前是 Linux 的默认线程库，相比 LinuxThreads 提供了更好的性能和兼容性，改进了线程的创建、调度和同步机制，减少了用户态线程库对内核的依赖，提升了多线程程序的性能和可靠性。

和 LinuxThread 一样，NPTL 也基于一对一线程模型，使用 clone() 系统调用创建轻量级进程实现。但其之所以被称为内核级线程库，是因为其实现依赖内核的特殊支持。NPTL 不再使用 manager thread，而是将一部分管理职责托管给内核，比如给所有线程发送信号等。

NPTL 中线程的同步不再依赖信号，转而使用一个新的线程同步原语 fast userspace mutex (futex)。futex 基于共享内存，通过对内存的原子操作来检测是否有竞争发生，没有竞争时不需要陷入内核，有竞争时需要执行系统调用来 wait 或 wake-up。

NPTL 利用了线程组的概念，给线程组中的任何一个线程发送信号时整个线程组能收到信号。

NPTL 还利用了内核中改进的信号处理、调度算法、线程局部存储、分层的内核锁和增强的内存管理等。todo

> ### Next Generation Posix Threading Package (NGPT) 
>
> IBM 为了改进 LinuxThreads 启动的项目，使用多对多的两极线程模型，和 NTPL 是竞争关系，在 2003 年停止开发；大约同一时间，Redhat Linux 9 中发布了最初的 NPTL。

> Main 函数执行到最后并返回时，会调用 `exit` ，给所有线程发送信号来终止进程，但如果提前终止主线程，例如调用`pthread_cancel`，整个进程并不会被终止，其他线程会照常运行。 

### 内核线程

（这里指的不是内核级线程，而是内核中运行的一些特殊线程）在内核线程被引入之前，内核任务委托给一些周期性执行的进程。内核线程就是内核的分身，一个分身可以处理一件特定事情。这在处理异步事件如异步IO时特别有用。内核线程的使用是廉价的，唯一使用的资源就是内核栈和上下文切换时保存寄存器的空间。

创建内核线程的 kernel_thread() 函数本质上还是调用 do_fork，传递 `CLONE_VM|CLONE_UNTRACED` flag。

与普通进程区别：

- 内核线程只运行在内核态，而普通进程既可以运行在内核态，也可以运行在用户态。
- 内核线程只使用大于 PAGE _OFFSET 的线性地址空间（因为内核线程只运行在内核态）。而普通进程可以用4GB的线性地址空间。

典型内核线程：

- *idle*: pid 为 0，是所有进程的祖先
- *init*: pid 为 1，由进程 0 创建，执行 init 函数后由内核线程变为一个普通进程
- *kthreadd*: pid 为 2，由进程 0 创建，用于管理和调度其他内核线程
- *keventd*, ...





> ### Inverview question: Process vs Thread
>
> - Context Switch: Switching processes is super expensive:
>   - Process switching uses an interface in an operating system, while swithing/creating threads doesn't require a kernel call.
>   - Process switching involves:
>     - Loading the corresponding Process Control Block (PCB) from the PCB table (in kernel stack). 
>     - Loading CPU state info (registers) and memory management info (segmentation tables, page tables) from PCB.
>     - Flush Translation Lookaside Buffer (TLB), to ensure correct virtual-pyhsical address translation
>   - Thread switching involves much smaller context: 
>     - thread program counter, registers, stack pointers
> - Communication: Threads in the same process share the same virtual memory space, so thread communication can be as simple as sharing a variable or object.However, processes's memory are usually isolated, and requires Inter Process Communication (IPC), which is not very efficient. 
> - Isolation: Processes are usually isolated, while a crash in a thread takes down the whole process.
>
> 一次起500个线程 会怎样？崩溃？
