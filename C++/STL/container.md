# STL Containers

> All STL containers store a copy of the inserted data!!

### allocator

Dynamically manage memory for stl containers. 考虑到小型区域可能造成内存破碎问题，SGI STL设计了双层级配置器，第一层配置器直接使用malloc()和free()，第二层配置器则视情况采用不同的策略：当配置区块超过128bytes时，调用第一层级配置器，当配置区块小于128bytes时，采用复杂的memory pool方式。

### array (fixed size array)

A `std::array` is a very thin wrapper around a C-style array. It has friendly value semantics, so that it can be passed to or returned from functions by value. Its interface makes it more convenient to find the size, and use with STL-style iterator-based algorithms.

`std::array`与其他容器最大的不同是，其元素直接存放在实例内部而不是堆上。`std::array`既可以作为函数返回值，又可以作为编译期常量。**它是编译期返回集合数据的首选**。

`std::array`相比于内建数组几乎没有额外开销，但更安全，可读性和可维护性更高，应当尽量使用。 `std::array` 不会隐式转成指针（需显式调用` data()` ），可以方便地按值传递、按值返回、赋值。C++14~17 中 std::array 逐渐变得比内建数组更适合配合 constexpr，C++20中swap, sort等都constexpr了，编译期的计算正变得愈加容易。

### vector (dynamic array)

##### reserve vs resize

reserve: **only** affect **capacity**, not affact size, won't initialize any instances

Resize: will insert or delete elements to the vector to make it given **size** (could call constructor!)

##### Capacity Growth

The capacity grows by double or 1.5 times of the previous size. Every time a vector's capacity grows the elements may need to be copied and all the itrators will becom invalid.

##### Boolean vector: `std::vector<bool>`

`std::vector<bool>`的`operator[]`不返回`bool&`，而是返回`std::vector<bool>::reference`对象。这是因为`std::vector<bool>`规定了使用一个打包形式（packed form）表示它的`bool`，每个`bool`占一个*bit*，但是C++禁止对`bit`s的引用，所以返回一个**行为类似于**`bool&`的对象`std::vector<bool>::reference`.

### list (doubly linked list)

No random access, only bidirectional iteration.

`push_back` puts a new element at the end of the `vector` 

`insert` allows you to select new element's position.

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



### unordered_map, unordered_set (hash map)



### Constness of set and maps

`set` is using a **const iterator** because: Elements form a tree that accelerates operations on the set, thus all elements must be const to keep the constraints of the underlying tree.

A key of a `unordered_map` or `map` is also `const`, which means the type of `pair` is actually `std::pair<const KEY, VAL>`. Unfortunately, if you iterate throught them via `std::pair<KEY, VAL>`, a copy and an emplicit conversion will be triggered, thus introduce overhead. Solution is elegant- use `auto`: `for(const auto& p : m)`.



# Iterator

### begin，cbegin，rbegin

分别为普通迭代器，只读迭代器，还有反向迭代器。降序排列元素可向 `std::sort()` 传送反向迭代器。

使用时，优先考虑`const_iterator`而非`iterator`；编写通用型代码时优先考虑非成员函数 `std::begin` 以支持原生数组等类型。



# Algorithms

> \#include <algorithm>

### swap: 

？？？quick sort

### find

### sort

 ```c++
template< class RandomIt >
void sort( RandomIt first, RandomIt last );
// or
template< class RandomIt, class Compare >
constexpr void sort( RandomIt first, RandomIt last, Compare comp );
 ```

 a comparator like this needed:

 ```
bool compare(const MyClass& o1, const MyClass& o2);
// or 
 
 ```

### for_each

Applies the given function to the result of dereferencing every iterator in the range [`first`, `last`). The function's **return result is ignored**.

```c++
std::vector<int> v{3, -4, 2, -8, 15, 267};
std::for_each(v.cbegin(), v.cend(), [](const int& n) { std::cout << n << ' '; });
std::for_each(v.begin(), v.end(), [](int &n) { n++; });
```

### transform: 

applies the given function to a range and **stores the function result** in another range, keeping the original elements order and beginning at 3rd paremeter, 

 - can be either in-place modify or non-inplace.
 - can be either unary operation or binary operation

 ```c++
 std::string s{"hello"}; // replace-write to the same location
 std::transform(s.cbegin(), s.cend(), s.begin(), 
                [](unsigned char c) { return std::toupper(c); });
 std::vector<std::size_t> ordinals; // insert at back
 std::transform(s.cbegin(), s.cend(), std::back_inserter(ordinals),
         [](unsigned char c) { return c; });
 ```

  可以认为功能大于等于 for_each，因为函数的返回值也被保存下来。注意如果不是尾部插入，则要求容器中已经有相应数量的元素用于被覆盖，否则可能会 segmentation fault.

