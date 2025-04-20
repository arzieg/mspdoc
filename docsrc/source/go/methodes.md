# Methodes




## Definition
An **interface** specifies abstract behavior in terms of **methods**

```go
type Stringer interface { // in "fmt"
    String() string
}
```

*Concrete* types offer methods that *satisfy* the interface. 

Without interfaces we would have to write many functons for many concrete types, possible coupled to them

Worst Case:
```go
func OutputToFile(f *File) ....
func OutputToBuffer(b *Buffer) ...
func OutputToSocket(s *Socket ...)
``` 

Better - we wanto to define our function in terms of *abstract behavior*

```go
type Writer interface {
    Write([]byte) (int,error)
}

func OutputTo (w io.Writer, ...)
```

Interface <--- Method ---  Object

An interface specifies required bahavior as a **method set**

Any type that implements that method set satisfies the interface. This is known as structural typing ("duck" typing). No type will declare itself to implement ReadWriter explicitly

In go we see the interface from the consumer side not the producer side. 


A **method** is a special type of function
In go: it has a **receiver** parameter *before* the function name parameter:

```go
type IntSlice []int
func (is IntSlice) String() string {
    ...
}
```

Terms: 
    is'a = substitution
    has'a = composition

A method may be defined on any **user-declared** (named) type. 
That means methods can't be declared on int, but

```go
type MyInt int

func (i MyInt) String() string {
    ...
}
```
The same method name may be bound to different types

## Receivers

```go
package geometry
import "math"

type Point struct{ X, Y float64 }
    // traditional function
    func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
    }

    // same thing, but as a method of the Point type
    func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
    }

p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // "5", function call
fmt.Println(p.Distance(q)) // "5", method call, p.Distance aka selector
```

## Signature

All methods of a given type must have unique names, but different types can use the same name for a method, like the Distance methods for Point and Path;

```go
// A Path is a journey connecting the points with straight lines.
type Path []Point
// Distance returns the distance traveled along the path.
func (path Path) Distance() float64 {
    sum := 0.0
    for i := range path {
        if i > 0 {
        sum += path[i-1].Distance(path[i])
        }
    }
    return sum
}
```

-> Methode: path.Distance und (s.o) p.Distance (point.Distance). Die eine Methode erwartet ein Slice, die andere zwei Punkte. 

## Value & Pointer Receiver

We distinguish between:

1. value Receiver (no change or original, took a copy)
2. pointer Receiver (change on the original)

typically we us first letter of the type name for the receiver (p):

```go
type Point struct {
    X, Y float64
}

func (p Point) Offset(x, y float64) Point {
    return Point{p.x+x, p.y+y}
}

func (p *Point) Move(x,y float64) {
    p.x += x
    p.y += y
}
```



## Interfaces and substitution

var w io.Writer
var rwc io.ReadWriteCloser

w = os.Stdout  // OK, os.File has Write method
w = new(bytes.Buffer)  // OK, *bytes.Buffer has write method
w = time.Second // Error, no Write method

rwc = os.Stdout // OK, os.File has all 3 methodes
rwc = new(bytes.Buffer) // Error no close methods
w = rwc  // OK. io.ReadWriteCloser has Write
rwc = w  // Error, no close method

the *receiver* must be of the right type (pointer or value):

```go

type IntSet struct { ... }
func (*IntSet) String() string
var _ = IntSet{}.String()   // Error, String needs *IntSet (l-value)

var s IntSet
var _ = s.String  // OK, s is avariable &s used automatically

var _ fmt.Stringer = &s // ok, Pointer to something
var _ fmt.Stringer = s // Error, no String method because it will a pointer receiver

```

## Composition

io.ReadWriter is actually defined by go as two interfaces: 

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
type ReadWriter interface {
    Reader
    Writer
}
```

Basic idea in go: keep the interface small and flexible



## Interface declaration

All methodes for a given type must be declared in the same package where the type is declared
This allows a package importing the type to know all the methods at compile time.
But we can alway extend the type in a new package through embedding.

