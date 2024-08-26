# C++ Memory Layout & Management

the following segments are sorted from high address to low address (in **virtual memory**):

- Kernel space: shared, map to same physical address for every process

- Stack segment: local variable (automatic, continuous memory, grow from high to low address)

- Heap segment: dynamic storage (large pool of memory, not contiguous, programmer controlled e.g. `new`, grow from low to high address).

 > Heap allocation algorithm:
 >
 > - Free list: use a linked list to store free blocks in the heap
 > - Bitmap: divide the heap into blocks, use 2 bits for each block to indicate its status
 > - Object pool

- Block Started by Symbol segment: **uninitialized** global variable and static variable (init to 0) `.bss`

- Data segment: **initialized** global variable and static variable `.data`

- Read-only data segment:  constant global variable, string literal, virtual pointer table `.rodata`

- Code (text) segment: codes `.text`



# C Memory Allocation

> Originally defined in C Library's \<string.h>. Use \<cstring> when writing C++, which put everything in namespace `std`

动态内存开辟主要有 malloc、calloc、realloc 几种，通过系统调用 brk 和 mmap 从操作系统申请内存。

- `malloc(size)`：分配给定字节数大小的空间，不初始化
- `calloc(num,size)`：在返回在堆区申请的那块动态内存的起始地址之前，会将每个字节都初始化为0，接受类型大小和个数两个参数（历史原因需要两个参数，不用管，反正malloc一般也是 num*size 来用）
- `realloc(ptr,new_size)`：重新分配之前由 `malloc`、`calloc` 或 `realloc` 分配的内存块的大小，可扩容或截断
- `alloca`：在栈上分配内存，有些平台不支持

一般小块内存分配会使用 `brk` 或 `sbrk` 调整堆的边界，大块内存请求则调用 `mmap`。阈值默认为 128KB，可通过 `MMAP_THRESHOLD` 修改。

调用 malloc 时，并不会真正申请物理内存，不会影响 RSS。以 brk 调用为例，其仅移动 `brk` 指针，一个表示进程虚拟内存空间中 `堆空间` 的顶部的指针。物理内存的申请延迟到对虚拟内存进行读写的时候，通过缺页异常来实现。

内存操作：

- `memcpy(dest,src,count)`
- `memset(dest,c,count)`: use fill byte `c` to fill a memory of length `count`

> Never use `memcpy` to copy classes. (e.g. copy unique_ptr)
>
> Never use `memset` to reset classes. (e.g. vptr set to zero, resulting nullptr error) (`bzero` is deprecated)

???

C++使用全局new或delete可以很轻松的操控内存，但也很容易引起内存破碎。防止内存破碎的一个方法就是对每个类重载new和delete，从不同固定大小的内存池中分配不同类型的对象。



# OS difference

### POSIX standard

- `malloc`: xxx, use `free` to release memory
- `posix_memalign`: 针对页大小对齐 The address of the allocated memory will be a multiple of *alignment*, which must be a power of two and a multiple of `sizeof(void *)`. use `free` to release memory

### Sun Solaris

- memalign: allocates size bytes on a specified alignment boundary and returns a pointer to the allocated block. The value of the returned address is guaranteed to be an even multiple of alignment. The value of alignment must be a power of two and must be greater than or equal to the size of a word.

### Windows

- _aligned_malloc



# new/delete vs malloc/free

- `malloc/free`: 
  - STL function of C/C++
  - only allocate memory, no initialization
  - if success, return `void*` pointers, require casting
  - if fail, return null pointer

- `new/delete`: 
  - **operator** of C++ (can even be overridden to private)

  - Allocate memory and also call **default** constructors/ free memory and call destructors


  > require default constructor, but can use "allocator" to call other constructors if default one doesn't exist 

 - if success, return pointers of the corresponding type; if fail, throw exception

### placement new: 

允许向 new 传递额外的地址参数，从而在预先指定的内存区域创建对象。可用于在栈上创建对象，缓解堆上开辟内存的开销

 ```c++
new (place_address) type
new (place_address) type (initializers)
new (place_address) type [size]
new (place_address) type [size] { braced initializer list }
 ```

### new[] / delete[]:

For creating arrays of instances. 若是自定义的数据类型，用new []申请的空间，必须要用delete []来释放，因为delete []会逐一调用对象数组的析构函数，然后释放空间，如果用delete，则只会调用第一个对象的析构函数，但空间还是会被释放

### std::nothrow

`std::nothrow_t`an empty class type used to disambiguate the overloads of throwing and non-throwing allocation functions(`new`). `std::nothrow` is a constant of this type. Usage: `int *p = new (std::nothrow) int[100];` 此种重载在分配失败时不抛出异常而是返回空指针



# Smart Pointers

智能指针包裹原始指针，其行为看起来像原始指针，但避免了很多陷阱。几乎原始指针能做的所有事情智能指针都能做，而且出错的机会更少。前身是 `auto_ptr`，有许多问题, deprecated from C++11, removed from C++17, not recommended to use.

### Unique pointer

Allows exactly one owner of the underlying pointer. 相当于对内存的独占，所以为只可移动类型，不支持拷贝赋值和拷贝构造，支持移动赋值和移动构造。

`unique_ptr`可以轻松高效的转换为`shared_ptr`：

支持自定义的删除器。删除器类型是`unique_ptr`类型的一部分。使用默认删除器时，`unique_ptr`对象和原始指针大小相同。自定义删除器可以实现为函数或者*lambda*时，尽量使用*lambda*，因为函数指针形式的删除器会使`unique_ptr`的大小从一个字（*word*）增加到两个。

### shared pointer & weak pointer

##### shared_ptr

Reference-counted smart pointer. The raw pointer is not deleted until all `shared_ptr` owners have gone out of scope or have otherwise given up ownership.

默认资源销毁是通过`delete`，通过构造函数创建时支持自定义删除器和分配器，而通过 `make_shared` 创建时只支持自定义分配器。有别于`unique_ptr`，删除器类型不是`shared_ptr`类型的一部分。

##### weak_ptr

A `weak_ptr` provides access to an object that is owned by one or more `shared_ptr` instances, but does not participate in reference counting. (no ownership) 类似`std::shared_ptr`但不影响对象引用计数

 - `weak_ptr::use_count()`: get the use_count of the resource

 - `weak_ptr::expired()`: Equivalent to use_count() == 0. The destructor for the managed object may not yet have been called, but this object's destruction is imminent (or may have already happened).

 - `weak_ptr::lock()`: if not expired, create a temporary shared_ptr to access the resource, otherwise return a default-constructed (empty) `shared_ptr<T>`

   > 另一种形式是以`weak_ptr`为实参构造`shared_ptr`，如果`weak_ptr`过期，会抛出一个`std::bad_weak_ptr`异常。

##### control block

`shared_ptr` 和 `weak_ptr` 在底层使用了control block 控制块，用于统计引用计数和销毁资源。根据创建方式为 Ctor 或 `make_shared`, 有两种不同的控制块实现，后者性能和安全性较高，具体可参考 STL 源码笔记 [STL/source-code-note.md](STL/source-code-note.md) 中关于 `<memory>` 的部份。

持有关系：`shared_ptr` 持有两个指针，一个指向目标对象，一个指向 control block； control block 持有目标对象指针，分配器，删除器(仅第一种实现)，以及两个引用计数, *shared_owners*和 *share_weak_owners* 。

对象销毁：当 *shared_owners* 减到0时，使用删除器(第二种实现时，使用分配器)销毁目标对象

控制快销毁：当 *shared_owners* 和 *share_weak_owners* 都减到0的时候，拷贝分配器成员到栈上，销毁分配器成员，再使用 `deallocate` 销毁 control block (类似delete this)，最后离开函数时栈上的分配器随栈自动回收。

##### overhead

- `shared_ptr` 对象的大小 (2 words) 是 `unique_ptr` 对象 (1 word) 的两倍，因为要保存控制块指针
- control block 最小只占用 3 个 word，自定义 deleter 和 allocator 可能会让它变大一点
- 涉及引用计数的修改，需要承担一两个原子操作的开销
- 对`shared_ptr`解引用的开销不会比原始指针高

##### shared_ptr aliasing constructor (C++20)



### Construct Smart Pointer: `std::make_unique` & `std::make_shared`

优先考虑使用`std::make_unique`和`std::make_shared`，而非直接使用`new` 后传入

##### 好处1：避免潜在内存泄漏

对于这种函数调用 `foo(std::shared_ptr<Widget>(new Widget), bar());`, 编译器可能会先 `new Widget`再执行`bar()`，然后调用 `shared_ptr` Ctor。此时如果`bar()`产生了异常，那么动态分配的`Widget`就会泄漏。

##### 好处2：性能高

使用`std::make_shared`允许编译器生成更小，更快的代码，并使用更简洁的数据结构。如果 `new` 后再传给 `shared_ptr` Ctor，程序实质上会调用两次 `new`，分别为对象和控制块分配内存。且控制块中需要包含额外的簿记信息。具体可参考 STL 源码笔记 [STL/source-code-note.md](STL/source-code-note.md) 关于 `<memory>` 的部份。



### dynamic_pointer_cast

如果基类的指针指向派生类，想用派生类独有的函数，就会使 std::dynamic_pointer_cast ，但只可作用于 shared_ptr，可用于unique_ptr，因为c++标准库觉得此做法违背了 unique_ptr 的唯一性。



### Function parameter passing semantics (C++ Core Guideline R32-R35)

- `void pass(unique_ptr<T>)` : pass ownership, recommended by 32 but no  by vital

- `void share(shared_ptr<T>)`: share ownership

- `void pass(unique_ptr<T> &&)` : pass ownership, should be always better than pass-by-value because a move ctor is saved.

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

### GNU C Allocator

`malloc` is defined in C standard, may be implemented by syscall `brk` and `mmap`. 

GNU C Library, *glibc*'s  `malloc` is derived from `ptmalloc` (pthreads malloc) (`ptmalloc` is based on Doug Lea's `dlmalloc`) , 并针对多线程环境进行了优化。

*glibc* memory allocator talks to the OS kernel, request and release the virtual memory for the processes in a wise way. (e.g. request a large virtual memory from the OS, and allocate them to processes eventually)

### C++ Allocator

 in C++, the `new` and `delete` operators are typically implemented on top of the C library functions `malloc` and `free`. 

### C++ STL Allocator

See STL/core/allocator.md