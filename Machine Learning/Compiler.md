解决两个问题：

1. 计算图优化（前端）
2. 在不同的硬件上部署模型（后端）





# TVM

TVM是一个端到端的机器学习编译框架，它的目标是优化机器学习模型让其高效运行在不同的硬件平台上。
它前端支持TensorFlow, Pytorch, MXNet, ONNX等几乎所有的主流框架。
它支持多种后端(CUDA,ROCm,Vulkan,Metal,OpenCL,LLVM,C,WASM)及不同的设备平台(GPU,CPU,FPGA及各种自定义NPU)。

基本步骤：

1. 导入其他框架的模型 
2. 将该模型转换成Relay（TVM的高层级IR）. Relay支持特性：
   - 传统数据流式的表示 - 函数式语言风格的表示
   - 两种风格的混合 Relay 能够进行图层次的优化 
3. 将Relay转换成更细粒度的Tensor Expression(TE) Relay使用FuseOps 将模型划分成小的子图，在此过程中可以使用一些schedule原语进行优化（如tiling, vectorization, parallelization, unrolling, and fusion） TOPI包含一些预定义的常用Operator。
4. 编译生成Tensor IR (TVM的低层级IR，相对于Relay）。 TVM支持的后端包括:
   - LLVM, 通过它可以生成llvm支持的所有硬件如x86, ARM.
   - 特定编译器，如NVCC, NVIDIA的编译器.
   - 通过BYOC(Bring Your Own Codegen)框架实现
5. 编译生成机器代码。 TVM可以将模型编译成可链接的对象模块来通过轻量级的运行时来运行。 它提供多种语言的支持。TVM也支持将模型和运行时统一打包。

# XLA