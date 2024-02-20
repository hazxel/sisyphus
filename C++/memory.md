# C Memory Alloocation

动态内存开辟主要有 malloc、calloc、realloc 几种

- malloc：分配给定字节数大小的空间
- calloc：在返回在堆区申请的那块动态内存的起始地址之前，会将每个字节都初始化为0，接受类型大小和个数两个参数
- realloc：重新分配之前由 `malloc`、`calloc` 或 `realloc` 分配的内存块的大小
- `alloca`：在栈上分配内存，有些平台不支持

调用 malloc 时，并不会真正申请物理内存，不会影响 RSS，只是移动 `brk` 指针，即一个表示进程虚拟内存空间中 `堆空间` 的顶部的指针。而且物理内存的申请延迟到对虚拟内存进行读写的时候，通过缺页异常来实现。

# C++ Memory Layout & Management

the following segments are sorted from high address to low address (in **virtual memory**):

- Kernel space: map to same physical address for every process

- Stack segment: local variable (automatic, continuous memory, grow from high to low address)

- Heap segment: dynamic storage (large pool of memory, not contiguous, programmer controlled e.g. `new`, grow from low to high address).

 > Heap allocation algorithm:
 >
 > - Free list: use a linked list to store free blocks in the heap
 > - Bitmap: divide the heap into blocks, use 2 bits for each block to indicate its status
 > - Object pool

- BSS (Block Started by Symbol) segment: **uninitialized** global variables and static variables (init to 0)

- Data segment (read-only): **initialized** global variables, static variable, **virtual pointer table**

- Code (text) segment: codes



C++使用全局new或delete可以很轻松的操控内存，但也很容易引起内存破碎。防止内存破碎的一个方法就是对每个类重载new和delete，从不同固定大小的内存池中分配不同类型的对象。



# OS difference

### POSIX standard

- `malloc`: xxx, use `free` to release memory
- `posix_memalign`: 针对页大小对齐 The address of the allocated memory will be a multiple of *alignment*, which must be a power of two and a multiple of `sizeof(void *)`. use `free` to release memory

### Sun Solaris

- memalign: allocates size bytes on a specified alignment boundary and returns a pointer to the allocated block. The value of the returned address is guaranteed to be an even multiple of alignment. The value of alignment must be a power of two and must be greater than or equal to the size of a word.

### Windows

- _aligned_malloc





# Smart Pointers

### Unique pointer

赋值：由于unique_ptr对于内存的独占特性，unique_ptr不支持直接的赋值操作，而只能支持右值引用的赋值

拷贝构造&移动构造：不支持拷贝构造，只支持移动构造

### shared pointer、unique pointer 与 中间层

shared_ptr在底层使用了两个技术，一个是引用计数，另一个是引入了一个中间层。为了管理目标对象，所创建的中间层被称为manager object。其中除了目标对象的裸指针，还有两个引用计数。一个用于shared_ptr，一个用于weak_ptr。当shared count减到0的时候，managed object就会被销毁。只有shared count和weak count都减到0的时候，manager object才会被销毁。

Smart Pointers are used to help ensure that programs are free of memory and resource leaks and are exception-safe. In most cases, when you initialize a raw pointer or resource handle to point to an actual resource, pass the pointer to a smart pointer immediately.

- `unique_ptr`: Allows exactly one owner of the underlying pointer.
- `shared_ptr`: Reference-counted smart pointer. The raw pointer is not deleted until all `shared_ptr` owners have gone out of scope or have otherwise given up ownership.
- `weak_ptr`: Special-case smart pointer for use in conjunction with `shared_ptr`. A `weak_ptr` provides access to an object that is owned by one or more `shared_ptr` instances, but does not participate in reference counting.
 - expired: Equivalent to use_count() == 0. The destructor for the managed object may not yet have been called, but this object's destruction is imminent (or may have already happened).
- `auto_ptr`: deprecated from C17, not recommended to use

Implementation:

- Counting references
- Destructor: when instance is out of scope, the destructor is called automatically

```c++
#include <memory>
```



### dynamic_pointer_cast

如果基类的指针指向派生类，想用派生类独有的函数，就会使 std::dynamic_pointer_cast ，但只可作用于 shared_ptr，可用于unique_ptr，因为c++标准库觉得此做法违背了 unique_ptr 的唯一性。



### Function parameter passing semantics (C++ Core Guideline R32-R35)

- `void pass(unique_ptr<T>)` : pass ownership

- `void share(shared_ptr<T>)`: share ownership

- `void pass(unique_ptr<T> &&)` : pass ownership

- `void pass(shared_ptr<T> &&)`: pass a share of ownership

- `void reset(unique_ptr<T> &)`: modify the `unique_ptr`   itself (reset/release/swap)

- `void reset(shared_ptr<T> &)`: modify the `shared_ptr`   itself (reset/release/swap)

- const reference: maybe not, consider raw pointers or references

  

### Avoid smart pointers for general usage (C++ Core Guideline F.7)

Any function that does not manipulate lifetime should take raw pointers or references instead. Reasons:

- smart pointer add restrictions to caller function
- passing shared pointer may introduce run-time overhead
- `const unique_ptr<int>&` doesn't change ownership, but requires a particular ownership of the caller
- Function arguments naturally live for the lifetime of the function call, and so have fewer lifetime problems.

Bad example: function `f` only use the object, lifetime not used at all

```c++
void f(shared_ptr<widget>& w) { use(*w); };

shared_ptr<widget> my_widget = /* ... */;
f(my_widget); // ok
widget stack_widget;
f(stack_widget); // error, f is restricted to only accepting shared_ptr<widget>
```

Good example:

```c++
void f(widget& w) { use(w); };

shared_ptr<widget> my_widget = /* ... */;
f(*my_widget);
widget stack_widget;
f(stack_widget); // ok -- now this works
```

### Thread Safe

smart pointers are not thread safe.



# Allocator

*glibc* memory allocator talks to the OS kernal, request and release the virtual memory for the processes in a wise way. (e.g. request a large virtual memory from the OS, and allocate them to processes eventually)