## Boost

Boost library are header files (usually under `/usr/include/boost`), install using `boost-devel` using package manager.

 



MP11: mp_for_each ???





mixin（boost::noncopyable, non-copyable mixin）？？？mixin不一定是template？

```cpp
class MyType : private NoCopySemantics {
  ...
};
```