# GPU

GPU并不是一个独立运行的计算平台，而需要与CPU协同工作，可以看成是CPU的协处理器。因此“GPU并行计算”，其实是指 CPU+GPU 的异构计算架构。在该架构中，GPU 与 CPU 通过 PCIe 总线相连并协同工作。

GPU包括更多的运算核心，其特别适合数据并行的计算密集型任务，如大型矩阵运算，而CPU的运算核心较少，但是其可以实现复杂的逻辑运算，因此其适合控制密集型任务。另外，CPU上的线程是重量级的，上下文切换开销大，但是GPU由于存在很多核心，其线程是轻量级的。因此，基于CPU+GPU的异构计算平台可以优势互补，CPU负责处理逻辑复杂的串行程序，而GPU重点处理数据密集型的并行计算程序，从而发挥最大功效。

### CUDA Core

SP (Streaming Processor) 流处理器是GPU最基本的处理单元，在 fermi 架构开始被叫做CUDA core。一个SP一次可以执行一个thread。

CUDA Core 一般包含多个数据类型，每个数据类型包含多个小核心，比如图中的 INT32 Core 和 FP32 Core 就各有 4×16 个，在计算专用卡上还可能会包含 FP64 Core（比如 V100 和 A100 显卡）

### SM (Streaming MultiProcessor)

 一个 SM 由多个 CUDA Core组成，每个SM根据GPU架构不同有不同数量的 CUDA Core，Pascal 架构中一个 SM 有 128 个 CUDA Core。

SM 采用 SIMT(Single-Instruction, Multiple-Thread，单指令多线程) 架构，以 warp  (线程束) 为最小单元创建、管理、调度和执行线程。WarpSize 是由硬件决定的，目前CUDA的 WarpSize都是 32。

- warp 内的 thread 以不同数据资源执行相同的指令，它们从同一个程序地址开始执行，但各自有自己的指令地址计数器和寄存器状态，因此也可以自由地分支和独立执行。
- 一个 warp 一次执行一条公共指令，因此当一个 warp 的所有 32 个线程都同意它们的执行路径时，就可以实现完全的效率。但如果 warp 内的线程通过依赖于数据的条件分支发散，则 warp 需要依次执行每个分支路径，并禁用不在该路径上的线程。（但是比如在只有八个SP的情况下只能同时跑8个threads，轮替执行，这时仅仅是软件层面的抽象而让上层看起来是32个同时执行）
- 一个warp中的线程必然在同一个block中，如果block所含线程数目不是 WarpSize 的整数倍，就会剩余一些 inactive 的thread，它们也会消耗SM资源产生浪费。
- 一个 SM 可同时处理多个 warp，透过 warp 的切换来隐藏 thread 的延迟、等待，以实现大规模并行。

### RT Core (Ray Tracing Core)

### Tensor Core

Tensor Core 直译为张量核心，是 GPU 上一块较为独立的计算单元。它使用半精度（FP16）作为输入和输出，使用全精度（FP32）进行中间结果计算从而不损失过多精度（不是**网络层面**既有 FP16 又有 FP32）。GPU 上有 Tensor Core 是使用混合精度训练加速的必要条件。

### Memory

每个线程有自己的私有本地内存（Local Memory），而每个 Block 都有共享内存（Shared Memory）,可以被 Block 中所有线程共享，生命周期与线程块一致。此外，所有的线程都可以访问 Grid 内的全局内存（Global Memory）、常量内存（Constant Memory）以及纹理内存（Texture Memory）。

shared memory 会通过 bank（现在硬件：32 个， 4-byte 1区分） 管理， 同一 bank 内的存取会有 bank conflicts，无法并发。所以在内存读取的时候需要考虑怎样安排计算顺序，让不同 thread 可以操作不同 bank 的数据，增加并行度。



# CUDA (Compute Unified Device Architecture)

CUDA 是 NVIDIA 推出的通用并行计算平台和编程模型，提供了GPU编程的简易接口，以利用 NVIDIA 的 GPU 进行图像处理之外的运算，支持 C/C++，Python，Fortran等语言。

CUDA中，CPU 及其内存被称为主机端（host），GPU及其内存被称为设备端（device）。典型的CUDA程序的执行流程如下：

1. 分配 host 内存，并进行数据初始化；
2. 分配 device 内存，并从 host 将数据拷贝到 device 上；
3. 调用 CUDA Kernel 函数在 device 上完成指定的运算；
4. 将 device 上的运算结果拷贝到 host 上；
5. 释放 device 和 host 上分配的内存。

### CUDA 软件堆栈

CUDA提供了三种不同的API，调用层级由底层到高级依次为：

- CUDA Driver：Driver API 从底层控制 GPU，包括访问GPU硬件资源、配置GPU寄存器和指令流等。CUDA Driver API的编程复杂，但有时能通过直接操作硬件的执行实行一些更加复杂的功能键，或者获得更高的性能。由于它使用的设备端代码是二进制或者汇编代码，因此可以在各种语言中调用。CUDA Driver API被放在nvCUDA包里，函数前缀为 `cu`。
- CUDA Runtime：CUDA Runtime 是对 CUDA Driver 封装的上层接口库，内部调用 CUDA Driver 的接口。Runtime API 函数可用于分配和释放设备上的内存、将数据从主机复制到设备并执行核函数等任任务。CUDA Runtime API 的编程较为简洁，通常都会用这种 API 进行开发，其 API 被打包放在 CUDAArt 包中，函数前缀为 `cuda` 。
- CUDA Library：是对 CUDA Runtime API 更高层的封装，主要包含一些成熟的高效函数库
  - cuBLAS：基本的矩阵与向量运算库，提供了与BLAS相似的接口，可以用于简单的矩阵计算。
  - cuSPARSE：线性代数库，包含一些通用稀疏线性代数函数，支持一系列稠密和稀疏的数据格式。
  - cuFFT：利用GPU进行傅立叶变换的函数库，提供了与广泛使用的FFTW库相似的接口。
  - cuDNN(NVIDIA CUDA Deep Neural Network)： 用于深度神经网络的 GPU 加速原语库，为标准例程（如前向和后向卷积、池化、规范化和激活层）提供了高度调优的实现。
  - cuDPP：提供了很多基本的常用的并行操作，如排序、搜索等，帮助快速搭建并行计算程序。

总结：CUDA Runtime 已经足够满足大部份应用的需求，并不需要使用 CUDA driver API。

> CUDA Toolkit (nvidia)： CUDA完整的工具安装包，其中提供了 Nvidia 驱动程序、开发 CUDA 程序相关的开发工具包等可供安装的选项。包括 CUDA 程序的编译器、IDE、调试器等，CUDA 程序所对应的各式库文件以及它们的头文件。
> CUDA Toolkit (Pytorch)： CUDA不完整的工具安装包，其主要包含在使用 CUDA 相关的功能时所依赖的动态链接库。不会安装驱动程序。

### NVCC

NVCC 是CUDA的编译器，是 CUDA Toolkit 中的一部分。由于 CUDA 程序有两种代码，一种是运行在cpu上的host代码，一种是运行在gpu上的device代码，所以 nvcc 要保证两部分代码能够编译成二进制文件在不同的机器上执行。

NVCC  能将自家的 CUDA C-语言（不支持完整的C语言标准），也就是执行于GPU的部分编译成PTX中间语言或是特定 NVIDIA GPU 架构的机器代码（device code）；而执行于 CPU 部分的 C/C++代码（host code）仍依赖于外部的编译器，如 GCC，MSVC 等。CUDA 通过函数类型限定词区别 host 和 device 上的函数：

- `__global__`：定义核函数。从 host 中调用，在 device 上执行，可带参数，但返回类型必须是`void`，不支持可变参数，不能成为类成员函数。注意用`__global__`定义的 kernel 是异步的，这意味着host 不会等待 kernel 执行完，而会直接执行下一步。
- `__device__`：在device上执行，仅可以从device中调用。这些函数不是kernel，它们通常用于在 GPU 上执行的计算中重用代码或实现一些辅助功能，可被`__device__`或`__global__`函数调用。
- `__host__`：在host上执行，仅可以从host上调用，一般省略不写

> `__global__`不可以和`__device__`或`__host__`限定词同时使用，但`__device__`和`__host__`可以同时使混用，此时函数会在device和host都编译。

> #### NVCC JIT (Just-In-Time) 即时编译 ???

### Kernel

Kernel 是在GPU上并行执行的函数或代码片段。标记为 `__global__`的函数是核函数。它们会被编译成GPU的指令集，然后在GPU上执行。

CUDA 的线程并行主要分三个层级：

- grid 网格：CUDA 线程结构的第一层次。grid 包括了一个 kernel 所启动的所有线程，它们共享相同的全局内存空间。grid由多个 block 组成。
- block 线程块：由多个 thread 组成
- thread 线程：CUDA 中最小的执行单位

对 kernel 的调用：

```c++
__global__ void kernel(param list) {}
kernel<<<Dg,Db,Ns,S>>>(param list);
```

三个尖括号 `<<< >>>` 表明是对 cuda kernel 的调用，其中的 kernel 参数告诉编译器用多少个线程来执行：

- `Dg`，指定 grid 的维度和尺寸，为 `dim3` 类型
- `Db`，指定 block 的维度和尺寸，为 `dim3` 类型
- `Ns`，指定每个 block 在静态分配的 shared memory 以外最多能分配的 shared memory大小 (byte)
- `S` ？？？

`dim3` 基于整数向量类型 `unit3` ，可以看成是包含三个无符号整数成员（x，y，z）的结构体，特殊点在于在定义时的缺省值初始化为 1。因此 grid 和 block 可以灵活地定义为 1-dim，2-dim 或 3-dim 结构。

在 Kernel 内部，可以访问以下内置变量：

- `gridDim`: 表示 Grid 中 Block 的维度，为 `dim3` 类型
- `blockIdx`: 表示当前 BLock 在 Grid 内部的位置，为 `uint3` 类型
- `blockDim`: 表示 Block 中的 thread 维度，为 `dim3` 类型
- `threadIdx`: 表示线程在 Block 内部的位置，为 `unit3` 类型

### 同步

`__sync_threads`：CUDA 中非常关键的一个同步原语，a block level synchronization barrier，即当同一个 Block 内的线程执行至此函数时，会等待全部的线程到达此处后再继续执行。

`cudaDeviceSynchronize`：同步当前设备上的所有CUDA流。 它会阻塞调用它的线程，直到所有设备上的CUDA流都执行完为止。 这可以确保在进行后续的CUDA操作时，先前的操作已经完成。

### CUDA 中的内存分配

为了在 GPU 上计算，我需要分配 GPU 可访问的内存，而CUDA 中的统一存储器可提供一个系统中所有 GPU 和 CPU 都可以访问的内存空间。

- `cudaMalloc`: CUDA版的`malloc`函数，用于在设备上开辟一段内存。使用后需要使用`cudaFree`释放。
- `cudaHostAlloc`/`cudaMallocHost`: 分配的内存只能由 host 访问，必须使用额外的数据传输函数才能将数据传输到设备端
- `cudaMallocManaged` 在统一内存中分配数据，它返回一个指针， host 和 device 都可访问。使用效果与cudaMalloc 类似，区别在于cudaMallocManaged 分配的空间会使用 UM 系统自动调度，一般搭配cudaMemPrefetchAsync使用。
- `cudaMemcpy`: 用于在主机和设备之间同步数据。根据 src 和 dst 为 host 或 device，有四种版本。程序运行到这一步之后会进入阻塞状态，直到同步完成。
-  `cudaFree` 释放指针。





# Mixed Precision

GPU 上有 Tensor Core 是使用混合精度训练加速的必要条件。





# 数据并行

数据并行是将数据划分为多个小数据，发送到不同处理节点上，使用相同的模型参数进行计算，将计算的不同结果进行汇总后获得最终结果。

### ZeRO (Zero Redundancy Optimizer)

 ZeRO 是一种用于减少数据并行策略下的显存占用的方法。在普通的数据并行策略中，每个 GPU 都独立地维护一组完整的模型参数，计算与通信效率较高，但内存效率较差。这个问题在训练大型模型时尤为突出。ZeRO 可以有效地减少显存消耗量，这意味着在同样的显存下，可以训练更大的模型。

##### 三阶段

ZeRO分为三个阶段，分别对应 O、P 和 G。每个 GPU 仅保存部分 OPG，三个阶段逐级递加：

- 阶段1，优化器状态分区
- 阶段2，添加梯度分区优化
- 阶段3，添加参数分区优化



# 集合通信

https://zhuanlan.zhihu.com/p/493092647#:~:text=%E9%9B%86%E5%90%88%E9%80%9A%E4%BF%A1%EF%BC%88Collective%20Communications%EF%BC%89%E6%98%AF,%E5%8E%9F%E8%AF%AD%EF%BC%8C%E6%AF%94%E5%A6%82%EF%BC%9A1%E5%AF%B9



# GPU Partitioning

### Multi-Stream

通过创建多个 Stream，增加 Kernel 执行的并行度，提高资源利用率

### Multi-Process Service (MPS)

一个逻辑上的分区方式，将 SM 分组并分配给不同的进程使用，使多个进程同时在一张卡上跑（不是切时间片）

### Multi-Instance GPU (GIG)

在硬件上将 SM 进行分割，每个 partition 具有独立的 shared memory 带宽，保证不同进程之间的隔离性。



