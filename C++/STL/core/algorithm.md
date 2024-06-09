

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