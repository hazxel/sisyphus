

# Algorithms

> \#include <algorithm>

### swap

Implementation: (in \<type_traits\>)

```c++
template<typename T>
void swap(T &a,T &b) noexcept {
    T temp = std::move(a);
    a = std::move(b);
    b = std::move(temp);
}
```

### find, find_if, find_if_not

```c++
if (std::find(v.begin(), v.end(), n) == std::end(v)) { /* do sth */ }
if (std::find_if(w.begin(), w.end(), [](int i){ return i%2==0; }) != w.end()) {}
```

### sort

 ```c++
std::sort(s.begin(), s.end());
std::sort(s.begin(), s.end(), std::greater<int>());
struct { bool operator()(int a, int b) const { return a < b; }} customLess;
std::sort(s.begin(), s.end(), customLess);
std::sort(s.begin(), s.end(), [](int a, int b){ return a > b; });
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

### copy_if



### remove & remove_if

If you don't actually need a new copy, `remove` and `remove_if` will removes the elements from the original container.

```c++
auto noSpaceEnd = std::remove(str1.begin(), str1.end(), ' ');

```

