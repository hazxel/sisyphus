# Generator

在生成器函数中，返回值使用yield语句而不是 return 语句。变成generator的函数，在每次调用`next()`的时候执行，计算出下一个元素的值，并通过`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行，直到计算到最后一个元素，抛出`StopIteration` 异常。

```python
def genSeq(max):
		for i in range(max):
				yeild i
# usage
seqIter = genSeq(100)
print(seqIter.__next__()) # 使用成员 next 方法
print(next(seqIter))			# 使用非成员 next 方法
for n in seqIter: 				# for 循环直接遍历
  	print(n)
```

生成器表达式是用圆括号来创建生成器，其语法与推导式相同，只是将 [] 换成了 () 

> python2中分range()和xrange() 其中range()是生成一个list，是一个列表生成式 而xrange()是一个生成器 
>
> python3中取消了xrange(), 内置的range()实际就是python2里的xrange()，返回生成器



# 数据结构

### tuple

有序不可修改。可查询、切片操作。用`()`表示

### set

无序可修改。用 `set()` 或 `{}` 初始化非空 set，用 `set()` 初始化空 set

### dict

无序可修改。用 `{}` 表示

字典解包：`**`, 可将一个 dict 解包为多个 kv pair 传递给函数，key 为形参，value 为实参

### list

有序可修改。通过索引进行查找、切片。用 `[]` 表示

列表解包：`*` ，可将一个 list 解包为多个参数传递给函数，如 `tasks = [task1, task2, task3]` ，则调用`asyncio.gather(*tasks)` 等价于 `asyncio.gather(task1, task2, task3)`

### numpy.array

N 维数组（矩阵）对象，可用 list 初始化：`np.array([1, 2, 3])`

##### 切片

单个维度的情况：

- `[i]`: 将返回单个元素 
- `[i:]` / `[:i]`: 返回 $i$ 项之后/之前所有项（包括 $i$ 项在内）
- `[i:j]`: 提取 $[i,j)$​​ 项。
- `[i:j:s]`: 按步长为 $s$ 提取 $[i,j)$ 间的项
- `[:]`: 提取所有项
- `[::s]`: 从索引为 0 的元素开始按步长 $s$​​ 取元素

多维情况：

- `[:,:]`: 逗号分隔
-  `[...,i:j]` / `[i:j,...]`: 表示之前/之后的所有维度都取全部(`:`)
- `a.shape`: 获取矩阵的各维度长度
- squeeze: 移除长度为一的维度，若指定了 axis 且有长度不为一的维度则会报错
  - `np.squeeze(a)`: Remove all axes of length one from array `a`.
  - `np.squeeze(a, axis=n)`: Remove $n_{th}$​ axes if length is 1
  - `np.squeeze(a, axis=(x,y,z))`: Remove all axes in tuple if lengths all equal to 1

##### VS list

- 一个 list 可以存放多种类型的数据，但是numpy数组中存放的数据类型必须全部相同
- list中实际上存放的是数据的地址非数据本身，如列表 `a=[1,2,3,4]` 需要4个指针和4个数据；但`a=np.array([1,2,3,4])` 只需要存放4个数据，节省内存，读取性能好

### namedtuple

tuple subclasses with named fields, 

- `_fields`: get a tuple of field name strings
- `.xxx`: access data by fileld name (like dictionary, but not `['key']`-way)

### heapq






# 闭包

闭包是指在一个函数内部定义另一个函数，并且该内部函数可以访问外部函数的变量和参数，即使外部函数已经执行完毕，内部函数仍然可以使用这些变量和参数。 在Python中，闭包是一种特殊的函数，它可以在函数内部定义另一个函数，并且返回该函数。

闭包的作用域：Python中变量的作用域有四种：local、enclosing、global和built-in。在一个函数内部定义另一个函数时，内部函数可以访问外部函数的变量和参数，这些变量和参数的作用域是enclosing。

生命周期：Python中变量的生命周期由变量的作用域和垃圾回收机制共同决定。当一个函数执行完毕后，如果其内部函数仍然在使用外部函数的变量和参数，那么这些变量和参数就不会被回收，它们的生命周期会延长，直到内部函数也执行完毕才会被回收。



# Decorator

装饰器是闭包的一种应用，用于拓展原来函数功能，这个函数的参数和返回值都是一个函数，使用python装饰器的好处就是在不用更改原函数的前提下增加新的功能。

 ```python
def logging(level):
		def outwrapper(func):
  			def wrapper(*args, **kwargs):
    				print("[{0}]: enter {1}()".format(level, func.__name__))
    				return func(*args, **kwargs)
    		return wrapper
  	return outwrapper


@logging(level="INFO")
def hello(a, b, c):
  	print(a, b, c)

hello("hello,","good","morning")
# [INFO]: enter hello()
# hello, good morning
 ```

 



# Syntax

### with

`with` 语句可用于简化资源管理，确保资源在使用完后能够正确地被释放。常见的用法是与文件操作或其他需要显式关闭的资源（如数据库连接、网络套接字等）一起使用：

```python
with open('example.txt', 'r') as file:
    content = file.read()
async with websockets.connect(uri) as websocket:
    await websocket.recv()
```

他们等同于使用 `try-finally` 块手动管理资源的打开和关闭：

```python
file = open('example.txt', 'r')
websocket = websockets.connect(uri)
try:
    content = file.read()
    await websocket.recv()
finally:
    file.close()
    websocket.close()
```





# 类

### 访问权限

- protected：以单下划线开头，也可使用@property decorator，但事实上任何人仍可访问，按照约定俗成的规定，“虽然我可以被访问，但是请把我视为私有变量，不要随意访问”
- private：以双下划线开头，访问会抛出 AttributeError

### 类变量 vs 实例变量：

- 实例变量出现在 `__init__` 里
- 类变量在 class 中， `__init__` 外

### 方法

##### Instance method: 

When creating an instance method, the first parameter is always `self`.  `self`是约定俗成的标识符，常作为实例方法的第一个参数，用于引用实例本身。它可以是任何标识符，但建议使用`self`以提高代码的可读性。

##### Class method: 

Pass the **class** as a first parameter, instead of the **instance**. This means that we don't need an instance at all, and we call the class method as if it was a static function. The first parameter is usually `cls`. 推荐使用  `@classmethod` 装饰器修饰。class method 可以访问类变量

##### Static method: 

static method 不接收任何 `self` 或 `cls`， 也不能访问类变量或实例变量，基本就只是一个在该类名空间下的普通函数。 推荐使用`@staticmethod` 装饰器修饰。

### 特殊变量/方法：

以双下划线开头，并且以双下划线结尾，任何人可访问，如：

- `__new__`：继承自object的类才有该方法，在实例创建之前被调用，任务是创建实例然后返回该实例对象，是个**静态**方法，至少要有一个参数cls（代表当前类），必须有返回值（即该实例对象）

- `__init__`：在实例创建完成后被调用，设置对象属性的一些初始值，通常用在初始化一个类实例的时候，至少有一个参数self(代表当前的实例)，无需返回值。除了不负责分配内存外，和 c++ 中的构造函数比较类似

- `__del__`：由垃圾回收器（通常是引用计数机制）调用。当对象的引用计数变为零时，`__del__` 方法可能会被调用，但具体的调用时机是不确定的，这一点和 c++ 中的析构函数有比较关键的区别

- `__call__`：使实例对象能够像函数一样被调用，即通过使用 `()` 运算符来调用。

- `__enter__` & `__exit__`: todo 

- `__aenter__` & `__aexit__`: todo

- `__class__`, `__slots__`, `__doc__`, `__iter__` ???

- `self` vs `cls`：

  > `self` is used for object methods
  >
  > `cls` is used for class methods

### property

`@property` ???



# Numpy

### array (见上文数据结构)



# Pandas

Pandas 可以简单、直观地处理关系型、标记型数据，如表格数据，时间序列数据等。

### 数据结构

- Series：带标签的一维同构数组，形似字典，包含索引（数据标签）和数据。由于基于一维 NumPy 数组实现，虽然其可以包含不同类型的数据，但可能会导致性能下降。
- DataFrame：大小可变的二维异构表格，可以看作用等长 Series 组成的字典

### 数据操作

- roling

- truncate

- set_index

- `itertuples`: 将 DataFrame 的每一行作为一个 namedtuple 返回 (若 `name=None` 则返回普通 tuple )。命名元组是具有命名字段的元组，这使得你可以通过属性名称访问字段。e.g. `tuple.px` or `tuple[0]`

- `iterrows`: 将 DataFrame 的每一行作为一个 (index, Series) 对返回，其中 `index` 是行的索引，`Series` 是行数据。

  > `itertuples` 在大多数情况下比 `iterrows` 更快，后者将每行都转换成一个 Series 对象。如果只需要读取数据但并不修改，`itertuples` 是一个更好的选择。繁殖如果需要对数据进行复杂操作或修改，`iterrows` 更适合。

- xxxx



# Multi-threading

### Thread (太低级!)

### 异步函数 `async` & `await`

异步函数是一种特殊类型的函数，它可以包含`await`表达式，并且在运行时可以挂起（暂停）和恢复执行。异步函数通过在 `def` 关键字前添加 `async`关键字来定义。

- `await` 关键字：只能出现在 `async` 修饰的函数中。将暂停当前异步函数的执行，并立即开始执行另一个异步操作，等待其操作完成后，恢复执行当前异步函数。；`await`关键字后的表达式通常是一个异步操作，比如调用另一个异步函数、调用返回`awaitable`对象的内置异步方法，或者调用一个带有异步支持的库函数。

- 直接调用 `async` 函数不会执行它（还会触发一个python warning），而是返回一个 `coroutine` 协程对象

-  `asyncio.run(func())` 挂起当前线程，执行异步调用，等待其返回后继续执行当前线程

  ```python
  import asyncio
  async def async_function():
      return 1
  async def await_coroutine():
      result = await async_function()   
      print(result) # 1
  asyncio.run(await_coroutine())
  ```

- `asyncio.create_task()` 创建一个异步任务，但并不启动线程执行它

- `asyncio.gather(*tasks)`  启动多个 task 并等待他们执行完毕

  ```python
  async def ws_channel_dump_tradeall_csv(inst_id_list):
      tasks = [asyncio.create_task(foo(arg)) for arg in args]
      await asyncio.gather(*tasks)
  ```

- xx

### coroutine

并发执行 example

```python
import concurrent.futures

with concurrent.futures.ThreadPoolExecutor(max_workers=40) as executor:
    to_do = []
    for arg in arg_list:  # 模拟多个任务
        to_do.append(executor.submit(some_task, arg))

    for future in concurrent.futures.as_completed(to_do):
        try:
            res = future.result() # catch potential exceptions in task
        except Exception as e: 
        		print("An error occurs: {}".format(e))
				print("Task result: {}".format(res))
```







# Python Interpreter

### Interpreter

Python 是一种解释性语言，执行时会先将 .py 文件中的源代码编译成 byte code，再由 Python Virtual Machine 来执行。这种机制与 Java、.NET 一致，但 Python 的 Virtual Machine 距真实机器的距离更远。

- `.py`: Python 源代码文件
- `.pyc`: `.py` 编译后生成的字节码文件，执行速度要远快于 `.py` 文件
- `.pyo`: 在优化模式下(`-O`) 编译`.py` 后生成的字节码文件，在 Python 3.5 后已经取消了优化模式。
- `.pyd`: basically a windows dll file，是 D 语言 (C/C++综合进化版本) 生成的二进制文件，比较难反编译
- `.whl` 文件: 是 Python 发布包的一种标准，本质上是一个 `.zip` 压缩包，包含了 `.py` 文件和元数据等。
  - 在不具备编译环境的情况下直接安装： `pip install xxxx.whl`
  - `pip install` 时带上参数 `--no-index --find-links /path/to/whl` 可避免联网，直接离线安装
  - 用 unzip 解压后，可以在 `xxxx.dist-info` 文件夹中的 `METADATA` 文件中看到依赖和版本等信息
  - `.whl` 文件名的后半部份，形如 *-cp36-abi3-linux_aarch64.whl* 的部份，在安装时会被用来判定平台系统是否支持此安装包
  
- xxxs



# Random

- `__name__`: if execute source file as main program, the interpreter sets it to `"__main__"`. 

  > Use `if __name__ == "__main__":` to protects users from accidentally invoking the script when they didn't intend to. (e.g. when `import`, the unprotected code will be executed)
  
- `import`: 会先查看当前目录，比如你有个文件叫 `random.py`，就会覆盖原生的 `random`库

- anaconda:

  - `conda env list`
  - `conda install`
  - `conda activate `
  - 安装包位置：`xxx/miniconda3/envs/envname/site-packages`

