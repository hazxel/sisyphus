# Build

### module vs package

Go module is a way to manage the dependencies of a Go project, introduced in v1.13. A go module is essentially a directory with two important files in the root: the `go.mod` file and the `go.sum` file.



### go.mod & go.sum

go.mod file only records the direct dependency's import path and version

> Any indirect dependency which is not listed in the **go.mod** file of your direct dependency or if direct dependency doesn’t have a go.mod file, then that dependency will be added to the **go.mod** file with `//indirect` as the suffix. . 

go.sum file lists down direct and indirect dependency required along with the version and checksum.



# Type

### Type assersion

A type assertion provides access to an interface value's underlying concrete value. (run-time error possibility)

```go
t := i.(T)
```

To test whether an interface value holds a specific type, a type assertion can return two values: the underlying value and a boolean value that reports whether the assertion succeeded.

```go
t, ok := i.(T)
```

### Type switch

A type switch is like a regular switch statement, but the cases in a type switch specify types (not values), and those values are compared against the type of the value held by the given interface value.

```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

### Array

#### Slice

- slice is a reference to an array
- append a slice will directly affect the underlying array, may overwrite the value

#### Range

When ranging over a slice, two values are returned for each iteration. The first is the index, and the second is a copy of the element at that index.

- You can skip the index or value by assigning to `_`.

  ```go
  for i, _ := range array
  for _, value := range array
  ```

- If you only want the index, you can omit the second variable.

  ```go
  for i := range array
  ```






# Struct

```go
type employee struct {
    string
    age    int
    salary int
}
```

#### Anonymous Field

A field having no name but only type.

#### Promoted Field

The first anonymous field. With promoted field, the fields and methods of this field will be “promoted” to the parent struct.





# Interface

- A type implements an interface by implementing its methods, without explicit declarations.

  ```go
  type I interface {
  	M()
  }
  ```

- Embedded interface:

  ```go
  type I2 interface {
  	I
    M2()
  }
  ```

- interface values can be thought of as a tuple of a value and a concrete type: $(value,type)$

  ```go
  var i I 				// default init to (<nil>, <nil>), cannot call M()
  var t *T
  i = t 					// now (<nil>, *main.T), can call M()
  i = &T{"hello"} // now (&{hello}, *main.T)
  ```

  - Go methods can be called with a nil receiver, so interface values with $nil$ underlying values can be a receiver
  - Calling a method on a nil interface is a run-time error because there is no type inside the interface tuple to indicate which *concrete* method to call.

- The interface type that specifies zero methods is known as the *empty interface*, which can hold values of any type:

  ```go
  var i interface{}
  ```

- Built-in interfaces

  - fmt.Stringer
  - fmt.Error
  - io.Reader

### 



# Clauses

### Import

- `.`: Dot import provides the perks of using elements of a package without mentioning the name of the package and can be used directly.
- `_`: When importing a package, every init() function in that package will be executed. However, sometimes we don't need to import the entire package, but only want to run the init() functions. That's when we use: `import _` 
- Alias: creates an alias of a package name

### Defer

A defer statement defers the execution of a function until the surrounding function returns.

```go
func main() {
	defer fmt.Println("world")
	fmt.Println("hello")
}
```

Running this will result in "hello\<\\n\>world"

### Channel

Channels are a typed conduit through which values can be sent and received with channel operator, `<-`.

```go
ch := make(chan int)	// create a channel
ch <- v								// Send v to channel ch.
v := <-ch  						// Receive from ch, and assign value to v.
```

By default, sends and receives block until the other side is ready.

### Goroutines
A goroutine is a lightweight thread managed by the Go runtime. The following clause `go f(x, y, z)` starts a new goroutine running `f(x,y,z)`.





# Tag & Reflection

A tag for a field allows you to attach meta-information to the field which can be acquired using reflection.

```go
type User struct {
    Name string `json:"name" xml:"name"`
}
```

Accessing tags using reflection:

```go
u := User{"Bob"}
t := reflect.TypeOf(u)
field, found := t.FieldByName("Name")
if found {
  fmt.Printf("%q\n", field.Tag) 						// json:\"name\" xml:\"name\"
	fmt.Printf("%q\n", field.Tag.Get("json")) // name
}

```

Encode data to json:

```go
b, _ = json.Marshal(u)
fmt.Println(string(b)) // {"Name":"Bob"}
```

omitempty tag:

After adding a `,omitemnty` in the tag, the key itself will be omitted from the JSON object, if a field is set to it’s default value (e.g. 0 for int). Note that the empty value should **exist** to make this works.





