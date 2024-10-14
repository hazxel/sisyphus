# Iterator

### begin，cbegin，rbegin

分别为普通迭代器，只读迭代器，还有反向迭代器。降序排列元素可向 `std::sort()` 传送反向迭代器。

使用时，优先考虑`const_iterator`而非`iterator`；编写通用型代码时优先考虑非成员函数 `std::begin` 以支持原生数组等类型。



### reverse iterator

简言之，`reverse_iterator` 持有一个 normal `iterator`, 称为 `current`。其所有加减操作都调用 `current` 的反向操作。但容器只有一个冗余迭代器 `end()` 来标识结尾，`begin()` 为可解引用的迭代器而不是冗余迭代器。由于不想引入新的冗余迭代器来标识开头， `reverse_iterator` 解引用的结果被设计为返回 `current` 的下一元素。**这意味着 `reverse_iterator` 解引用的结果永远和其持有的 `current` 有一步的错位。**

> - 反向迭代器 `rend()` 持有的 `current` 是 正向迭代器 `begin()`
> - 反向迭代器 `rbegin()` 持有的 `current` 是 正向迭代器 `end()`
> - 反向迭代器解引用会返回 `*(current-1)`
> - 注意，对 `rend()` 或 `end()` 解引用都是 ub

- 成员 ` _Iter current;` 是一个 normal `iterator`, `_Iter` 为模版参数
- 方法  `operator--()` 执行 `++current`  返回 `*this`
- 方法 `operator++()` 执行 `--current`  返回 `*this`
- 方法 `operator*()` 返回 `*(current--)`
- 方法 `base()` 返回 `current`，注意由于错位的存在，这一方法返回的是正序情况下的下一元素的迭代器



