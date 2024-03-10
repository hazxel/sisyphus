# Range

A range can be loosely thought of a pair of iterators. It's more convenient to pass a single range object to an algorithm than separate begin/end iterators.

```c++
std::vector<int> v{/*...*/};
std::sort( v.begin(), v.end() );
ranges::sort( v ); // equivalant
```



# Range-v3

Range-v3 is a generic library that **augments** the existing STL with facilities for working with *ranges*. Range-v3 is a header-only library, works with C++14/17/20, and is the basis of the range support in C++20.

### pipeline

Having a single range object permits *pipelines* of operations connected by `|`. In a pipeline, a range is lazily adapted or eagerly mutated in some way, with the result immediately available for further adaptation or mutation. 

Lazy adaption is handled by views, and eager mutation is handled by actions.

### View

参考string_view，拷贝代价很低，直接传值不必传引用。

The below uses views to filter a container using a predicate and transform the resulting range with a function. Note that the underlying data is const and is not mutated by the views.

```c++
std::vector<int> const vi{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
using namespace ranges;
auto rng = vi | views::remove_if([](int i){ return i % 2 == 1; })
              | views::transform([](int i){ return std::to_string(i); });
// rng == {"2","4","6","8","10"};
```

### Action

In contrast, *actions* do their work eagerly, but they also compose. Consider the code below, which reads some data into a vector, sorts it, and makes it unique.

```c++
extern std::vector<int> read_data();
using namespace ranges;
std::vector<int> vi = read_data() | actions::sort | actions::unique;
```





# STL ranges(C++20)

Range 在 C++20 的时候被纳入了标准库, 但其功能比 Range-v3 弱很多。而虽然range-v3功能比较多，但是某些测试证明用range-v3写出来的代码很多时候性能比不上之前老式的循环风格，应该是编译器目前的优化并没有覆盖ranges采用的全部技术。 所以无论是C++20的ranges，还是ranges-v3，都不好用，一个功能经常不全，一个性能经常不高。可以等几年再用，估计要到c++26才好用。

但是如果要学源码是可以的，直接看gcc10中ranges的实现，含注释只有3000多行，可以学到concept结合constexpr if和模板编程的N多技巧，收获比直接用ranges写几行代码大十倍。