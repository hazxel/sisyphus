# memory (clang)

### `__shared_count`: shared owner counter

- data member `long __shared_owners_;`
- define `virtual void __on_zero_shared()`
- implement `__add_shared` and `__release_shared`, update `__shared_owners_` atomically

### `__shared_weak_count`: weak owenr counter

- inherit  `__shared_count`
- data member`long __shared_weak_owners_;`
- define `virtual void __on_zero_shared_weak()`
- implement `__add_weak` and `__release_weak`, update `__shared_weak_owners_` atomically

### `__shared_ptr_pointer`: Control Block for `shared_ptr` CTOR

- inherit `__shared_weak_count`

- use `__compressed_pair` to store `_Tp` (目标对象指针), `_Dp` (删除器), `_Alloc` (分配器)

  使用删除器销毁对象，使用分配器创建并销毁和控制块

- override `__on_zero_shared` and `__on_zero_shared_weak` to release resource

### `__shared_ptr_emplace`: Control Block for `make_shared` and `allocate_shared`

效率比前者高，构造只包含一次 `new` 内存分配调用，一次性为对象和控制块分配堆内存。同时，避免了对控制块中的某些簿记信息的需要，潜在地减少了程序的总内存占用。此外，由于对象和控制块被分配在相邻内存，可以更好的利用cache locality。

- inherit `__shared_weak_count`

- use `__compressed_pair` to store `_Tp` (目标对象指针), `_Alloc` (分配器)

  不需要删除器是因为对象和控制块都是由同一个`_Alloc` (分配器)统一创建
  
- implement `_Tp*get()` to pass target object pointer to `shared_ptr`

### `shared_ptr`：shared ownership of an object through a pointer

- data member `element_type*  __ptr_;`, `__shared_weak_count* __cntrl_`
- overload `operator->`, return the pointer `__ptr_`
- overload `operator*`, return the derefrenced pointer `*__ptr_`

### `weak_ptr`: non-owning ("weak") reference to an object

- data member `element_type*  __ptr_;`, `__shared_weak_count* __cntrl_`

