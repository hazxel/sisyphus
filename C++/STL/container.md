# STL Containers

> All STL containers store a copy of the inserted data!!

### allocator

Dynamically manage memory for stl containers. 考虑到小型区域可能造成内存破碎问题，SGI STL设计了双层级配置器，第一层配置器直接使用malloc()和free()，第二层配置器则视情况采用不同的策略：当配置区块超过128bytes时，调用第一层级配置器，当配置区块小于128bytes时，采用复杂的memory pool方式。

### Erasable 要求

所有 STL 容器都要求元素类型是完整类型并满足可擦除 (Erasable) 的要求，当然许多成员函数附带了更严格的要求。所谓可擦除，就是指一个对象可以被给定 allocator 销毁。注意，引用本质上是别名，不可被销毁，所以不符合可擦除原则。



# array (fixed size array)

A `std::array` is a very thin wrapper around a C-style array. 其相比于内建数组几乎没有额外开销，但更安全，可读性和可维护性更高，应当尽量使用。 

`std::array`与其他容器最大的不同是，其元素直接存放在实例内部而不是堆上(当然实例可能还是在堆上，重要的是实例地址就是元素实际存放的地址，访问不需要跳转到其他内存页，所以性能好)。

`std::array` 不会隐式转成指针（需显式调用` data()` ），按值传递、按值返回、赋值都方便而高效。

`std::array`可以作为编译期常量，**是编译期返回集合数据的首选**。C++14~17 中 std::array 逐渐变得比内建数组更适合配合 constexpr，C++20中swap, sort等都constexpr了，编译期的计算正变得愈加容易。

一般直接使用花括号 `{}` 进行 aggregate initialization.



# vector (dynamic array)

### reserve vs resize

reserve: **only** affect **capacity**, not affact size, won't initialize any instances

Resize: will insert or delete elements to the vector to make it given **size** (could call constructor!)

### Capacity Growth

The capacity grows by double or 1.5 times of the previous size. Every time a vector's capacity grows the elements may need to be copied and all the itrators will becom invalid.

### 成员函数

- `insert`: 使用迭代器插入，插入到给定迭代器的前面：

  - 插入单个：`v.insert(v.end(), element);`
  - 插入多个相同：`v.insert(v.end(), 123, 0);`

  - 插入起始迭代器之间所有元素：`v.insert(v.end(), target.begin(), target.end());`

- `back`: 传回最后一个元素的引用

- `emplace_back`: 原地使用构造函数构造 

### Boolean vector: `std::vector<bool>`

`std::vector<bool>`的`operator[]`不返回`bool&`，而是返回`std::vector<bool>::reference`对象。这是因为`std::vector<bool>`规定了使用一个打包形式（packed form）表示它的`bool`，每个`bool`占一个*bit*，但是C++禁止对`bit`s的引用，所以返回一个**行为类似于**`bool&`的对象`std::vector<bool>::reference`.





# list (doubly linked list)

No random access, only bidirectional iteration.

`push_back` puts a new element at the end of the `vector` 

`insert` allows you to select new element's position.



# forward_list (linked list)



# deque

In C++, the STL `deque` is a sequential container that provides the functionality of a double-ended queue data structure.

In a regular queue, elements are added from the **rear** and removed from the **front**. However, in a deque, we can insert and remove elements from both the **front** and **rear**. Deque also support random access of its elements.

Internally it maintains a double-ended queue of *chunks* of **fixed size**. Each chunk is a vector, and the queue (“map” in the graphic below) of chunks itself is also a vector.



# priority_queue (similar to heap)

`priority_queue<Type, Container, Functional>`

- Type 数据类型

- Container 底层使用的容器类型，必须是用数组实现的容器，如 vector, deque 等，不可是 list，默认 vector

  > The container must satisfy the requirements of SequenceContainer, and its iterators must satisfy the requirements of LegacyRandomAccessIterator, and must provide `front()`, `push_back()`, `pop_back()`.

- Functional 比较的方式。使用基本数据类型时，默认是大顶堆

### 初始化

- 传 Functional 类型：`priority_queue<T,Container<T>,Cmptor)> pq;`
- 传 lambda 或函数：`priority_queue<T,Container<T>,decltype(cmptor)> pq(cmptor);`
- 缺省：`priority_queue<int>` 等同于 `priority_queue<int,vector<int>,less<int>>`

### 成员方法：

`top`, `pop`, `push`, `emplace`



# stack

没有迭代器，无法直接遍历元素，如果一定需要遍历，可能需要考虑使用 `vector`。

### 成员方法：

`top`, `pop`, `push`, `emplace`



# pair

> \#include <utility>

### Special member function

- CopyCtor & MoveCtor: `defaulted`
- CopyAssignOperator is `deleted` if either type's `is_copy_assignable<T>::value` evaluates to false. 
- MoveAssignOperator is `deleted` if either type's `is_move_assignable<T>::value` evaluates to false. 

Since a key of a `unordered_map` or `map` is also `const`, which means the type of `pair` is actually `std::pair<const KEY, VAL>`. 这意味着遍历表类型容器时，由于 KEY 带有 const 修饰，pair 将是不可拷贝/移动赋值的，需要多加注意。



# Map, Set, Multiset, Multimap (red black tree)



### compare

需要一个方法判断 key 的大小关系，默认使用 `std::less<Key>`，will invoke `operator<` on type `T` unless specialized. 函数返回值为 `bool`，参数为两个 `const T&`，自定义方法参考下文关于自定义哈希的示例。



# unordered_map, unordered_set (hash map)

Map: `find`, `erase`



### hash

`std::hash<T>` 已为数字、枚举、指针、字符串，以及其他一部份 STL 类型的键值定义特化；对于其他键值类型，需要自定义 `std::hash<T>`  特化或者传递哈希 callable (返回值为 `size_t`，参数为 `const & T`)：

```c++
// #1: template specialization of std::hash
namespace std {
template <>
class hash<Foo>{
public:
    size_t operator()(const Foo &name ) const {/*...*/}
};
}; 
unordered_map<Foo, int> m; // usage

//#2: functor
struct my_hash_struct{ 
    size_t operator()(const Bar & b) const {/*...*/}
}; 
unordered_map<Bar, int, my_hash_struct> m; // usage

// #3: function
size_t my_hash_func(const Boo & b) {/*...*/}
unordered_map<Boo,int,function<size_t(const Boo&)>> m(10, my_hash_func); // usage

// #4: lambda
auto hash = [](const Goo &g){ return std::hash<int>{}(g.val); };
auto comp = [](const Goo &l, const Goo &r){ return l.val == r.val; };
unordered_map<Goo, double, decltype(hash), decltype(comp)> m(10, hash, comp); // usage
```

### equal_to

除了哈希，还需要判断元素是否相同。默认使用 `std::equal<T>`, will invokes `operator==` on type `T` unless specialised. 函数返回值为 `bool`，参数为两个 `const T&`，自定义的方法和上述自定义哈希的方法类似。



# Constness of set and maps

`set` is using a **const iterator** because: Elements form a tree that accelerates operations on the set, thus all elements must be const to keep the constraints of the underlying tree.

A key of a `unordered_map` or `map` is also `const`, which means the type of `pair` is actually `std::pair<const KEY, VAL>`. Unfortunately, if you iterate throught them via `std::pair<KEY, VAL>`, a copy and an emplicit conversion will be triggered, thus introduce overhead. Elegant `auto` solution: `for(const auto& p : m)`.



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

