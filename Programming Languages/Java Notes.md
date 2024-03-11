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

##### Profile-Guided Optimization (PGO)

PGO is a compiler optimization technique in computer programming that uses profiling to improve program runtime performance.

