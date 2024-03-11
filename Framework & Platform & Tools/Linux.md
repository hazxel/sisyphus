Unix哲学KISS：keep it simple, stupid。在Linux系统里，一切看上去十分复杂的逻辑功能，都用简单到不可思议的方式实现，甚至有些时候看上去很愚蠢。但仔细推敲，人们将会赞叹Linux的精巧设计，或许这就是大智若愚。




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

