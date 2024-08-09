# Allocator

STL 容器都带有默认 Allocator 参数：`template<class T, class Allocator = std::allocator<T>`, Allocator 最重要的四个接口为：

- `allocate`：配置空间，足以存储n个T对象
- `deallocate`：释放配置的空间
- `construct`： 调用对象的构造函数
- `destroy`： 调用对象的析构函数

对于 allocator，不同 STL 的实现也略有不同：

### SGI STL Allocator

##### std::allocator

std::allocator部分符合STL标准，它在文件 defalloc.h 中实现。但是SGI STL的容器并不使用它，也不建议我们使用，它存在的意义仅在于为用户提供一个兼容老代码的折衷方法，其实现仅仅是对new和delete的简单包装。

std::allocators don’t provide a resizing operation, thus may be less performant.

##### std::alloc

std::alloc 是SGI STL的默认配置器，它在`<memory>`中实现。他由两级空间配置器实现：

- 第一级配置器处理超过 128B 的区块，认为这种区块足够大，直接使用`malloc()`，`realloc`和`free()`；
- 第二级配置器处理小于 128B 的区块，使用自由链表实现了内存池。维护多个自由链表，每个链表管理一种特定大小的内存块。可减少频繁系统调用，减少内存碎片。

### GNU Allocator

1. 标准分配器 (`std::allocator`)

- 功能: 提供基本的内存分配和释放功能。
- 用途: 适用于大多数情况下的通用分配需求。

2. 调试分配器 (`__gnu_cxx::malloc_allocator` 和 `__gnu_cxx::debug_allocator`)

- 功能: 提供额外的调试支持，帮助检测内存泄漏、越界访问和双重释放等问题。
- 用途: 开发和调试阶段使用，以提高代码的健壮性和稳定性。

3. 并行分配器 (`__gnu_parallel::allocator`)

- 功能: 优化多线程环境中的内存分配和释放，提高并发性能。
- 用途: 在多线程应用程序中使用，以减少锁争用和提高内存分配效率。

### Clang Allocator

libc++ 的 std::allocator 提供了基本的内存分配、构造、销毁和释放功能。它遵循 C++ 标准，确保兼容性和一致性，其实现主要关注简洁性和标准兼容性。 libc++ 也支持在调试模式下进行内存检查，但这些功能通常通过其他工具（如 AddressSanitizer）来实现，而不是通过特殊的 allocator 实现。
