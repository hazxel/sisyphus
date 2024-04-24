# Generator

根据前面的元素推断后面的元素，一边循环一边计算的机制叫 generator。

变成generator的函数，在每次调用`next()`的时候执行，计算出下一个元素的值，并通过`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行，直到计算到最后一个元素，抛出`StopIteration` 异常。

生成器表达式是用圆括号来创建生成器，其语法与推导式相同，只是将 [] 换成了 () 

> python2中分range()和xrange() 其中range()是生成一个list，是一个列表生成式 而xrange()是一个生成器 
>
> python3中取消了xrange(), 内置的range()实际就是python2里的xrange()，返回生成器



# 基本数据结构

- tuple：有序不可修改。可查询、切片操作。用（ ）表示
- set：无序可修改。用set（[ ]）表示
- dict：无序可修改。用{ }表示
- list：有序可修改。通过索引进行查找、切片。用[ ]表示

> ### list VS numpy.array 
>
> - list可以存放不同类型的数据，比如int,str,bool,float,list,set等，但是numpy数组中存放的数据类型必须全部相同
> - list中每个元素的大小可以相同，也可以不同，因此不支持一次性读取一列，而numpy数组中每个元素大小相同，支持一次性读取一列。numpy对二维数组的操作更为便捷   
> - list中数据类型保存的是数据存放的地址，即指针而非数据，比如一个有四个数据的列表a=[1,2,3,4]需要4个指针和4个数据，增加了存储空间，消耗CPU，但是a=np.array([1,2,3,4])只需要存放4个数据，节省内存，方便读取和计算



# 闭包

闭包是指在一个函数内部定义另一个函数，并且该内部函数可以访问外部函数的变量和参数，即使外部函数已经执行完毕，内部函数仍然可以使用这些变量和参数。 在Python中，闭包是一种特殊的函数，它可以在函数内部定义另一个函数，并且返回该函数。

闭包的作用域：Python中变量的作用域有四种：local、enclosing、global和built-in。在一个函数内部定义另一个函数时，内部函数可以访问外部函数的变量和参数，这些变量和参数的作用域是enclosing。

生命周期：Python中变量的生命周期由变量的作用域和垃圾回收机制共同决定。当一个函数执行完毕后，如果其内部函数仍然在使用外部函数的变量和参数，那么这些变量和参数就不会被回收，它们的生命周期会延长，直到内部函数也执行完毕才会被回收。

> # Decorator
>
> 装饰器是闭包的一种应用，用于拓展原来函数功能，这个函数的参数和返回值都是一个函数，使用python装饰器的好处就是在不用更改原函数的前提下增加新的功能。
>
> ```python
> def logging(level):
>        def outwrapper(func):
>            def wrapper(*args, **kwargs):
>                print("[{0}]: enter {1}()".format(level, func.__name__))
>                return func(*args, **kwargs)
>            return wrapper
>        return outwrapper
> 
> @logging(level="INFO")
> def hello(a, b, c):
>       print(a, b, c)
> 
> hello("hello,","good","morning")
> -----------------------------
> >>>[INFO]: enter hello()
> >>>hello, good morning
> ```
>
> 





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

- `__new__`：继承自object的类才有该方法，在实例创建之前被调用，任务是创建实例然后返回该实例对象，是个静态方法，至少要有一个参数cls（代表当前类），必须有返回值（即该实例对象）
- `__init__`：是在实例创建完成后被调用的，设置对象属性的一些初始值，通常用在初始化一个类实例的时候，是一个实例方法，至少有一个参数self(代表当前的实例)，无需返回值
- `__class__`, `__slots__`, `__doc__`, `__del__`, `__enter__`, `__exit__`, `__iter__` ???

- `self` vs `cls`：

  > `self` is used for object methods
  >
  > `cls` is used for class methods

### property

`@property` ???





# Random

- `__name__`: if execute source file as main program, the interpreter sets it to `"__main__"`. 

  > Use `if __name__ == "__main__":` to protects users from accidentally invoking the script when they didn't intend to. (e.g. when `import`, the unprotected code will be executed)