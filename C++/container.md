# STL Containers

> All STL containers store a copy of the inserted data!!

### array (fixed size array)

A `std::array` is a very thin wrapper around a C-style array. It has friendly value semantics, so that it can be passed to or returned from functions by value. Its interface makes it more convenient to find the size, and use with STL-style iterator-based algorithms.

### vector (dynamic array)

##### reserve vs resize

reserve: **only** affect **capacity**, not affact size, won't initialize any instances

Resize: will insert or delete elements to the vector to make it given **size** (could call constructor!)

##### Capacity Growth

The capacity grows by double or 1.5 times of the previous size. Every time a vector's capacity grows the elements may need to be copied and all the itrators will becom invalid.

### list (doubly linked list)

No random access, only bidirectional iteration.

### Forward_list (linked list)

### deque

In C++, the STL `deque` is a sequential container that provides the functionality of a double-ended queue data structure.

In a regular queue, elements are added from the **rear** and removed from the **front**. However, in a deque, we can insert and remove elements from both the **front** and **rear**. Deque also support random access of its elements.

Internally it maintains a double-ended queue of *chunks* of **fixed size**. Each chunk is a vector, and the queue (“map” in the graphic below) of chunks itself is also a vector.

### Priority Queue (heap)

##### Container

The type of the underlying container to use to store the elements. The container must satisfy the requirements of SequenceContainer, and its iterators must satisfy the requirements of LegacyRandomAccessIterator. Additionally, it must provide the following functions with the usual semantics: `front()`, `push_back()`, `pop_back()`. The standard containers std::vector. Also, std::deque satisfy these requirements.

### pair

> \#include <utility>

### Map, Set, Multiset, Multimap (red black tree)

> set is using a **const iterator** because:
>
> A set is like a map with no values but only keys. Those keys are used for a tree that accelerates operations on the set, they cannot change. Thus all elements must be const to keep the constraints of the underlying tree from being broken.

### unordered_map, unordered_set (hash map)



# Iterator

### begin，cbegin，rbegin

分别为普通迭代器，只读迭代器，还有反向迭代器。降序排列元素可向 `std::sort()` 传送反向迭代器
