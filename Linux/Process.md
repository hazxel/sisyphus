# Process

###  process descriptor 进程描述符

###  

### create process：

##### 表层系统调用：

- fork: copy all the resources of father

  > Copy-on-write (COW): 为了降低开销，fork最初并不会真的产生两个不同的拷贝，因为在那个时候，大量的数据其实完全是一样的。写时复制是在推迟真正的数据拷贝。若后来确实发生了写入，那意味着parent和child的数据不一致了，于是产生复制动作，每个进程拿到属于自己的那一份，这样就可以降低系统调用的开销。所以有了写时复制后呢，vfork其实现意义就不大了。 

- vfork: parent and child share the same address space

- clone: a call with parameter (clone_flags). Can select partial resource of parent to share with child

##### Process Creation Details

进程的创建主要通过系统调用 `execl`, `execle`或`execv`，读取 ELF文件，把其中的代码和数据部分放到链接时指定的内存位置，然后从指定的开始地址执行代码。

1. 创建一个独立的虚拟地址空间
2. 读取可执行文件头，少部份 segment 直接加载进内存，其余 segment 建立虚拟空间与可执行文件的映射，后续通过缺页中断加载进内存。
3. 将CPU的指令寄存器设置成可执行文件的入口地址，启动运行

### process status

- R：running，教科书一般将正在CPU上执行的进程定义为RUNNING状态、可执行但是尚未被调度执行的进程定义为READY状态，这两种状态在linux下统一为 TASK_RUNNING状态。
- Z：僵尸状态，进程在退出的过程中，处于TASK_DEAD状态。 在这个退出过程中，进程占有的所有资源将被回收，除了task_struct结构（以及少数资源）以外。于是进程就只剩下task_struct这么个空壳，故称为僵尸。
- S (TASK_INTERRUPTIBLE)，可中断的睡眠状态。
- D (TASK_UNINTERRUPTIBLE)，不可中断的睡眠状态。
- T (TASK_STOPPED or TASK_TRACED)，暂停状态或跟踪状态
- X：退出状态，进程即将被销毁

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


