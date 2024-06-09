

# Functor

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

### compare operators

比较大小的操作符，如 `std::less<Key>`，will invoke `operator<` on type `T` unless specialized. 函数返回值为 `bool`，参数为两个 `const T&`，自定义方法参考上文关于自定义哈希的示例。

### equal operator

判断相等的操作符，如 `std::equal<T>`, will invokes `operator==` on type `T` unless specialised. 函数返回值为 `bool`，参数为两个 `const T&`，自定义的方法和上述自定义哈希的方法类似。