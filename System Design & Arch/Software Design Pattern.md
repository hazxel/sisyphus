### Singleton



### Factory



### Producer-Consumer



### CRTP (Curiously Recurring Template Pattern)

```c++
class Base { 
	virtual void fun1() { // do something};
}

class Derived : Base {
 	void fun1() { //do somthin }
}

// =======================

template<T>
class Base {
	void fun1() {
		static_cast<T*>(this)->fun1Impl();
	}
}

class Derived : Base<Derived> {
 	void fun1Imple() {}
}
```

