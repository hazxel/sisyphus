### Accessibility

- public: every class can access the element
- protected: only subclasses and classes in the same package can access the element
- default: only classes in the same package can access the element
- private: only this class can access the element




### Overloading

- Overloading is resolved statically, based on the static type of the receiver and the static type of the arguments.
- for the receiver, compiler look at receiver's static type's methods, using the static type of the arguemts, and pick the most "specific" method.
- Then, at run time, if the dynamic type has override that method, the overriding method will be applied.



Dynamic Overloading:

If using the dynamic keyword, the method having the most specific parameter among receiver and all receiver superclasses will be picked at run time.

```java
public static Matrix add(Matrix m1, Matrix m2) { 
	return (m1 as dynamic).add(m2 as dynamic);
}
```



### Abstract Class

Abstract classes cannot be instantiated. They are meant to be subclassed.

> ### Abstract class vs Interface (after Java 8)
>
> In the old times, all the methods in the interface are abstract methods. In java 8, interfaces are able to have default implementations and static methods, which makes it more like an abstract class. However, interfaces still can't have a "constructor", and can only have "public static final" member variables.
>
> - 抽象类和接口都不能够实例化，但可以定义抽象类和接口类型的引用。
> - 一个类如果继承了某个抽象类或者实现了某个接口都需要对其中的抽象方法全部进行实现，否则该类仍然需要被声明为抽象类。
> - 接口比抽象类更加抽象，因为抽象类中可以定义构造器，可以有抽象方法和具体方法，而接口中不能定义构造器而且其中的方法全部都是抽象方法。
> - 抽象类中的成员可以是 `private`、默认、 protected 、public的，而接口中的成员全都是public的
> - 抽象类中可以定义成员变量，而接口中定义的成员变量实际上都是常量
> - 有抽象方法的类必须被声明为抽象类，而抽象类未必要有抽象方法



### Generics 

##### Type Erasure

Java introduced generics in version 1.4,but for backwards compatibility, Sun did not want to change the virtual machine. So, generic type information is erased by compiler:

- C<T> is translated to C

- T is translated to its upper bound

  > 如果类型参数是有界的，则将每个参数替换为其第一个边界；如果类型参数是无界的，则将其替换为 `Object`

- Casts are added where necessary

This result in:

- run-time information missing:

  - generic types not allowed with `instanceof`
  - `class` object of generic types not available
  - arrays `[]`  of generic types not allowed

- name clash in overloading:

  ```java
  // Compile-time error:
  class Erasure<S, T> {
   	void foo(S p) {} 
   	void foo(T p) {}
  }
  // correct
  class Erasure<S, T extends Person> {
   	void foo(S p) {}
  	void foo(T p) {}
  }
  ```


- Static fields are shared by all instantiations of a generic class (e.g. Array<Integer> and Array<Boolean>)

##### Wildcards

A wildcard represents an unknown type:

```java
static void printAll(Collection<?> c) {
	for (Object e : c ) {
		System.out.println(e); 
  }
}
```

Constrained Wildcards: 有界的类型参数

A wildcard can be bounded to allow clients to decide on variance at the use site.

- Use-Site Variance: Covariance:

  ```java
  class Random<T> {
    T next() {}
    void initialize(T i) {}
  }
  Random<? extends Person> r = new Random<Student>();
  Person a = r.next( );
  r.initialize(new Student()); // contravariance uses not checked
  ```

- Use-Site Variance: Contravariance

  ```java
  class OutputChannel<T> {
    void write(T x) {}
    T lastWritten() {}
  } 
  OutputChannel<? super Student> o = new OutputChannel<Person>( );
  o.write( new Student());
  Person s = o.lastWritten(); // covariance uses not checked
  ```

Principle: **PECS (Producer Extends, Consumer Super)**



### Inner Class

Static Nested Class是被声明为静态（static）的内部类，它可以不依赖于外部类实例被实例化。而通常内部类需要在外部类实例化后才能实例化。非静态内部类会持有外部类的引用（为了使用外部类的变量等），从而导致GC可能回收不了这部分引用，导致OOM



### Synchronized

##### Synchronized instance method

Only one thread per instance of the class can execute this method.

##### Synchronized static method

Only one thread can execute inside a static synchronized method per class, irrespective of the number of instances it has.

##### Synchronized block within method



### String

##### String Pool

A special memory region where Strings are stored by the JVM. JVM can optimize the amount of memory allocated for them by storing only one copy of each literal String in the pool. This process is called *interning*. 

When we create a String via the new operator, it will be stored in its unique memory region. String Class has a method `intern()` that can put the specific string into the pool and return the interned string. 

JVM为了提高性能和减少内存开销，在实例化字符串常量时进行了优化。JVM在Java堆上开辟了一个字符串常量池空间（StringTable)，JVM 通过 ldc 指令加载字符串常量时会调用 `StringTable::intern` 函数将字符串加入到字符串常量池中。

> `String` 类型密码安全性问题: [知乎讨论](https://www.zhihu.com/question/36734157) [案例](https://heapdump.cn/article/3984488)
>
> 业务代码中有时会需要处理密码，这种一般推荐使用 `char` 而不是 `String` 类型。**String 类型是不可变的**，意味着一旦创建了字符串，在GC前除了反射没有方法可以清除内存中的字符串数据。如果内存被 dump，会造成密码泄露风险。
>
> 使用反射修改 value时，如果该String指向JVM常量池中的字符串，以反射方式修改会导致其他指向该常量的引用出错。
>
> 但如果使用 `char[]` 类型，则可以显式清除数据。

##### StringBuilder vs Stringbuffer

String is unmutable, which means andy operations will result in new instances. `StringBuffer` and `StringBuilder` are mutable and have better performance.

`StringBuffer` is thread-safe and synchronized whereas `StringBuilder` is not.



### Data Structure

##### Array

```Java
int[] array = new int[len];
int[] array = new int[] {1, 2, 3, 4};
Arrays.sort(array)； // ascending order
Arrays.sort(array, Collections.reverseOrder()); // descending order
List l = Arrays.asList(1,2,3,4); // just a wrapper, don't use directly, feed to actual lists 
Arrays.copyOf();
Arrays.copyOfRange();
Arrays.fill();
```



##### List



##### Priority Queue

```java
Queue<Integer> heap = new PriorityQueue<>();
PriorityQueue<Integer> minHeap = new PriorityQueue<>((a,b)->b-a);
```

##### Hash Map

在Java8之前的实现中是用链表解决冲突的，Java8之后如果碰撞冲突的元素超过某个限制(默认8)，则使用红黑树来替换链表。

Resize: Initializes or doubles table size. If null, allocates in accord with initial capacity target held in field threshold. Otherwise, because we are using power-of-two expansion, the elements from each bin must either stay at same index, or move with a power of two offset in the new table.



# Dependency shading

Dependency shading of Java is the process of including a dependency in your project and relocating its classes to a different Java package, often renaming the packages and rewriting all affected bytecode. This is typically done to avoid conflicts between the versions of dependencies used by your library and the versions used by the consumers of your library.

> e.g. your library uses v1.0 of `org.example.foo`, and the consumers of your library use v2.0 of `org.example.foo`. To avoid a conflict at runtime, you can shade the `org.example.foo` dependency in your library, relocating its classes to a different package, such as `com.mylibrary.org.example.foo`.

# View jar package symbols

```shell
jar tvf /root/.m2/repository/org/apache/hadoop/thirdparty/hadoop-shaded-avro_1_11/hadoop-shaded-avro-1.2.0-rc1.jar | grep Stringable
```







# Java Virtual Machine（JVM）

### Compilation

Use `javac` to compile `.java` source code to `.class` file which contains symbol table and bytecode. Then the byte code was compiled optimized, and run by Just In Time Compiler (JIT)

### Java Native Interface (JNI)



### Garbage Collection （GC）

##### Garbage Detection Algorithms

- Counting References: can't handle circular references
- **Reachability Analysis**

##### GC Roots

GC roots are starting points for Reachability Analysis of the garbage collector processes. All objects directly or indirectly referenced from a GC root are not garbage collected. 

- Class: Classes loaded by a system class loader; contains references to static variables as well
- Stack Local: Local variables and parameters to methods stored on the local stack
- Active Java Threads: All active Java threads
- JNI References: Native code Java objects created for JNI calls; contains local variables, parameters to JNI methods, and global JNI references



### Java Runtime Environment (JRE)

The Java Runtime Environment is a set of software tools which are used for developing Java applications. It is used to provide the runtime environment. It is the implementation of JVM. It physically exists. It contains a set of libraries + other files that JVM uses at runtime.



### Just-In-Time (JIT) compiler

The Just-In-Time (JIT) compiler is a component of the Java Runtime Environment (JRE) that improves the performance of Java applications at run time.

在部分商用虚拟机中（如HotSpot），Java程序最初是通过解释器（Interpreter）进行解释执行的，当虚拟机发现某个方法或代码块的运行特别频繁时，就会把这些代码认定为“热点代码”。为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各种层次的优化，完成这个任务的编译器称为即时编译器（Just In Time Compiler，下文统称JIT编译器）。

##### Profile-Guided Optimization (PGO)

PGO is a compiler optimization technique in computer programming that uses profiling to improve program runtime performance.

##### JIT 时间开销

- 解释器的执行，抽象的看是这样的：输入的代码 -> [ 解释器 解释执行 ] -> 执行结果
- JIT编译然后再执行的话，抽象的看则是：输入的代码 -> [ 编译器 编译 ] -> 编译后的代码 -> [ 执行 ] -> 执行结果

说JIT比解释快，其实说的是“执行编译后的代码”比“解释器解释执行”要快，并不是说“编译”这个动作比“解释”这个动作快。事实上，JIT编译比解释执行一次略慢一些，而要得到最后的执行结果还得再“执行编译后的代码”。
所以对只执行一次的代码，解释执行总是比JIT编译执行要快。对只执行一次的代码做JIT编译再执行，可以说是得不偿失。对只执行少量次数的代码，JIT编译带来的执行速度的提升也未必能抵消掉最初编译带来的开销。只有对频繁执行的代码，JIT编译才能保证有正面的收益。

##### JIT 空间开销

对一般的Java方法而言，编译后代码的大小相对于字节码的大小，膨胀比达到10x是很正常的。所有只有对执行频繁的代码才值得编译，如果把所有代码都编译则会显著增加代码所占空间，导致“代码爆炸”。这也就解释了为什么有些JVM会选择不总是做JIT编译，而是选择用解释器+JIT编译器的混合执行引擎。

##### 热点代码检测

- 基于计数器的热点探测：为每个方法（甚至是代码块）建立计数器，统计方法的执行次数，如果执行次数超过一定的阀值，就认为它是“热点方法”

 - 方法调用计数器

 - 回边计数器

- 基于采样的热点探测：虚拟机会周期性地检查各个线程的栈顶，如果发现某些方法经常出现在栈顶，那这个方法就是“热点方法”



# GraalVM

???

