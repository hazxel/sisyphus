# Allocator

STL 容器都带有默认 Allocator 参数：`template<class T, class Allocator = std::allocator<T>`, Allocator 最重要的四个接口为：

- `allocate`：配置空间，足以存储n个T对象
- `deallocate`：释放配置的空间
- `construct`： 调用对象的构造函数
- `destroy`： 调用对象的析构函数

### SGI STL Allocator

##### std::allocator

std::allocator部分符合STL标准，它在文件 defalloc.h 中实现。但是SGI STL的容器并不使用它，也不建议我们使用，它存在的意义仅在于为用户提供一个兼容老代码的折衷方法，其实现仅仅是对new和delete的简单包装。

std::allocators don’t provide a resizing operation, thus may be less performant.

##### std::alloc 

std::alloc 是SGI STL的默认配置器，它在`<memory>`中实现。他由两级空间配置器实现：

当配置区块超过128bytes时，视之为足够大，便调用第一级配置器，直接使用`malloc()`，`realloc`和`free()`；第二级配置器实现了内存池和自由链表，当配置器区块小于128bytes时，可以从内存池和自由链表中获取空间，减少系统调用，提升性能。（最终都是用`malloc()`，`realloc`和`free()`）