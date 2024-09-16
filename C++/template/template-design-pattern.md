# Curiously Recurring Template Pattern (CRTP)

The main usage of the CRTP includes:

- add a generic functionality to a particular class.

 ```c++
template <typename T>
struct GeneralFunctions {
  void multiply(double multiplicator) {
    T& underlying = static_Cast<T&>(*this);
    underlying.setValue(underlying.getValue() * multiplicator);
  }
  void abs(); // and more ...
}
 
struct Data : public GeneralFunctions<Data> {
  double getValue() const;
  void setValue(double value);
};
 ```

 why not non-member template functions? **CRTP shows the interface**.

- create static interface

 ```c++
template <typename T>
class Base {
public:
	void interface() {
		static_cast<T*>(this)->impl();
	}
private:
  Base(){}; // no one can access private constructor, except the friend classes - T
  friend T; // won't compile unless derived class is T (avoid typo)
};
 
class Derived : public Base<Derived> {
  void impl();
};
 
template <typename T>
void exampleUsage(const Base<T> &foo) { foo.interface(); }
 ```

 why not virtual method? Avoid the virtual calls here because the information of which class to use was **available at compile-time**. 



# Mixin

 Mixin is complementary to the CRTP. A Mixin template defines a generic behavior, and inherit from the type you wish to plug functionality onto, to isolate the generic functionality.

```c++
class Name { void print() const; };

template<typename Printable>
struct RepeatPrint : Printable { // this is a generic functionality
  explicit RepeatPrint(Printable const& printable) : Printable(printable) {}
  void repeat(unsigned int n) const {
    while (n-- > 0)
      this->print(); // names in template base classes ignored, won't compile w/o "this"
  }
};

template<typename Printable> // deduce template args, no need explicit specify
RepeatPrint<Printable> repeatPrint(const Printable &printable) {
  return RepeatPrint<Printable>(printable);
}

Name name("Eddard", "Stark");
repeatPrint(name).repeat(10);
```

In the above example, we could add a `repeat` method to the `Name` class. But the concept of repeatedly call the `print` method is something that could apply to other classes, like a `PhoneNumber` class that could also have a `print()` method.

Minxin can also be a pack:

```c++
template<class... Mixins>
sturct X : public Mixins... {
  X(const Mixins&... mixins) : Mixinx(mixins)... {}
}
```



