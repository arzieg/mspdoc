# Filelayout

* Typdeklarationen, Constanten und Methoden werden in einem go-File abgelegt
* Funktionen werden in einem anderem go-File abgelegt (analog c mit .c und .h?)

# Links
https://blog.mikesahari.com/posts/go-structs/





# Types

## Typenklassifizierung

Unterschieden wird zwischen:

**Basic Types**

* number
* strings
* booleans

**Aggregate Types**

* array
* struct

**Reference Types**

* pointers
* slices
* maps
* functions
* channels

**Interface Types**

## int

* int8 / uint8
* int16 / uint16
* int32 / uint32
* int64 / uint64
* int / uint  - Plattformabhängig, ob int32 oder int64. Wichtig int != int64 beim Vergleich, sind auf go-Sicht zwei verschiedene Typen

rune = int32
byte = int8
uintptr = Low Level Pointer, wird z.B. in der Verbindung mit C-Programmen eingesetzt (package unsafe)

## Floating Points

* float32
* float64

math.IsNaN tests whether its argument is a not-a-number value
math.NaN returns such a value

## Operators

in absteigendem Vorrang

```
*   /   %   <<  >>  &   &^
+   -   |   ^
==  !=  <   <=  >   >=
&&
||
```

```
&   bitwise AND
|   bitwise OR
^   bitwise XOR
&^  bit clean (AND NOT)
<<  left shift
>>  right shift
```

## complex

Go provides two sizes of complex numbers, complex64 and complex128, whose components are float32 and float64 respectively.

real(x) - reelle Zahl 
imag - imaginiäre Teil einer komplexen Zahl

## Bool

true
false

## Strings

A string is an immutable sequence of bytes.

rune =  UTF-8-encoded sequences of Unicode code points (runes)

len = returns the number of bytes (not runes) in a string
index = operation s[i] retrieves the i-th byte of string s, where 0 ≤ i < len(s).
utf8.RuneCountInString(s) = Länge der rune


The i-th byte of a string **is not** necessarily the i-th character of a string, because the UTF-8 encoding of a non-ASCII code point requires two or more bytes.

s[i:j] = Substring von i bis <j

s[i:]  = Starte ab Index i

s[:j]  = bis Index <j

s[:]   = kompletter String

+      = String concatination

Ein *raw string literal* steht zwischen zwei backquotes \` \`

### Standard Packages for manipulating Strings

* bytes   - bytes manchmal performanter wenn man strings zusammenbaut
* strings
* strconv - konvertiert string in /zu bool, integer, float
* unicode - Vergleichsfunktionen wie IsDigit, IsLetter, IsUpper, IsLower für runes


strings und bytes teilen sich mehrere ähnliche Funktionen

| string    | byte     |
| ---    | ---     |
|func Contains(s, substr string) bool |   func Contains(b, subslice []byte) bool|
|func Count(s, sep string) int  | func Count(s, sep []byte) int|
|func Fields(s string) []string  |func Fields(s []byte) [][]byte|
|func HasPrefix(s, prefix string) bool  | func HasPrefix(s, prefix []byte) bool|
|func Index(s, sep string) int  | func Index(s, sep []byte) int|
|func Join(a []string, sep string) string |   func Join(s [][]byte, sep []byte) []byte|s


## const

When a sequence of constants is declared as a group, the right-hand side expression may be omitted for all but the first of the group, implying that the previous expression and its type should be used again.

```go
const (
    a = 1
    b
    c = 2
    d
) => abcd = "1 1 2 2"
```

const Generator iota (aka enumeration, enum)

```
const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)  => Sunday = 0
```

Ein komplexeres Beispiel: 
```go
const (
_ = 1 << (10 * iota)
KiB // 1024
MiB // 1048576
GiB // 1073741824
TiB // 1099511627776
PiB // 1125899906842624
EiB // 1152921504606846976
ZiB // 1180591620717411303424
YiB // 1208925819614629174706176
)
```

## Array

* fixed size
* If an array’s element type is comparable then the array type is comparable too. Arrays unterschiedlicher Größe können nicht verglichen werden

```go
var q [3]int = [3]int{1,2,3}
q := [...]int{1,2,3}   // automatische definition
r := [...]int{99: -1}  // 100 Elemente, alle 0 initialisiert bis auf das letzt -1
```

## Slices

* Slices represent variable-length sequences whose elements all have the same type.
* A slice has three components: a pointer, a length, and a capacity.
* Multiple slices can share the same underlying array and may refer to overlapping parts of that array!

* The slice operator s[i:j], where 0 ≤ i ≤ j ≤ cap(s), creates a new slice that refers to elements i through j-1 of the sequence s, which may be an array variable, a pointer to an array, or another slice. The resulting slice has j-i elements.

* die Kapzität eines Slices [i:j] welches von einem Array abgeleitet wird ist cap(array)-i, die Länge ist i-j

* Unlike arrays, slices are not comparable, we cannot use ==. 
* Standard library provides bytes.Equal function. For all other types of slice we have to write our own function
* The only legal slice comparison is against nil. The nil slice has length and capacity zero
* if you need to test whether a slice is empty, use len(s) == 0, not s == nil.
```go
var s []int    // len(s) == 0, s == nil
s = nil        // len(s) == 0, s == nil
s = []int(nil) // len(s) == 0, s == nil
s = []int{}    // len(s) == 0, s != nil
```

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {

	var a []int
	j1, _ := json.Marshal(a)
	fmt.Println(string(j1)) // nil

	b := []int{}

	j2, _ := json.Marshal(b)
	fmt.Println(string(j2)) // []

}
```


* The built-in function make creates a slice of a specified element type, length, and capacity. The capacity argument may be omitted, in which case the capacity equals the length.

```go
make([]T, len)
make([]T, len, cap) // same as make([]T, cap)[:len]
```

* Under the hood, make creates an unnamed array variable and returns a slice of it; the array is accessible only through the returned slice. In the first form, the slice is a view of the entire array. In the second, the slice is a view of only the array’s first len elements, but its capacity includes the entire array. The additional elements are set aside for future growth.


* immer auf len und cap achten

```go
package main

import (
	"fmt"
)

func main() {
	a := [3]int{1, 2, 3}
	b := a[:1]

	fmt.Println("a=", a)
	fmt.Println("b=", b)
	c := b[0:2]
	fmt.Println("c=", c) // kein Problem, hätte man vlt. erwartet, weil b:=a[:1], aber die cap(b) == cap(a)

	fmt.Println(len(b))
	fmt.Println(cap(b))

	fmt.Println(len(c))
	fmt.Println(cap(c))

    d := a[0:1:1]  // [i:j:k] len=j-i, cap=k-i  <- hier wird explizit Kapazität mit definiert
}
```

Fun:
```go
func main() {
	a := [3]int{1, 2, 3}
	b := a[0:1]
	c := b[0:2]

	fmt.Printf("a[%p] = %v\n", &a, a)
	fmt.Printf("b[%p] = %[1]v\n", b)
	fmt.Printf("c[%p] = %[1]v\n", c)

	c = append(c, 5)
	fmt.Printf("a[%p] = %v\n", &a, a)  
	fmt.Printf("c[%p] = %[1]v\n", c)

} 
a[0xc000018018] = [1 2 3]
b[0xc000018018] = [1]
c[0xc000018018] = [1 2]
a[0xc000018018] = [1 2 5]  <- a  hat die cap von 3, c len=2,cap3. Ich ändere c, d.h. c[3] wird angehängt und ändere damit a :-)
c[0xc000018018] = [1 2 5]
```

besser, wenn man dieses Verhalten nicht haben möchte

```go
c:=a[0:2:2]   // len 2, cap 2

c= append(c,5)  <- nun zwinge ich c dazu dass eine reallocation stattfindet, so das c [1,2,5] enthält, a aber bei [1,2,3] bleibt
```

* Funktionsübergabe bei einem Slice. Wenn man am Slice etwas ändert muss auch immer ein Slice zurückgegeben werden, da es ggfs. durch 
Kapazitätserweiterung zu Verschiebungen des Speicherortes kommen kann. 

```go
func appendInt(x []int, y ...int) []int {   // x slice, y eine beliebige Anzahl weiterer int's, Rückgabe ein Slice
	...
}
```

A slice can be used to implement a stack. 

Given an initially empty slice stack, we can push a new value onto the end of the slice with append: `stack = append(stack, v) // push v`

The top of the stack is the last element: `top := stack[len(stack)-1] // top of stack`

and shrinking the stack by popping that element is `stack = stack[:len(stack)-1] // pop`


## maps

* It is an unordered collection of key/value pairs in which all the keys are distinct, and the value associated with a given key can be retrieved, updated, or removed using a constant number of key comparisons on the average, no matter how large the hash table.

* a map is a reference to a hash table

* map type is written map[K]V

* The key type K must be comparable using ==

* Definition:
```go
ages := make(map[string]int) // mapping from strings to ints

ages := map[string]int{}

ages := map[string]int{
	"alice": 31,
	"charlie": 34,
}
```

* Access
```go
ages["alice"] = 32
delete(ages, "alice") // remove element ages["alice"]
```

* wenn ein Key noch nicht vorhanden ist, wird es angelegt. Ist eine "safe operation", da value initial auf 0 gesetzt wird.

* Ein Map - Element kann aber nicht per Adresse angesprochen werden, da bei Erweiterung einer Map aka Hash-Table diese sich ändert, also &ages["bob"] geht nicht!

* Iteration over keys using range
```go
for name, age := range ages {
	fmt.Printf("%s\t%d\n", name, age)
}
```

* Check if key exist
```go
if age, ok := ages["bob"]; !ok { /* ... */ }  /* "bob" is not a key in this map; age == 0. */
```

* As with slices, maps cannot be compared to each other; the only legal comparison is with nil.
```go
func equal(x, y map[string]int) bool {
	if len(x) != len(y) {
		return false
	}
	for k, xv := range x {
		if yv, ok := y[k]; !ok || yv != xv {
			return false
		}
	}
	return true
}
```



## structs

* A struct is an aggregate data type that groups together zero or more named values of arbitrary types as a single entity. 
* Each value is called a field.
* Field order is significant to type identity.
* The name of a struct field is exported if it begins with a capital letter
* A struct type may contain a mixture of exported and unexported fields.
* A named struct type S can’t declare a field of the same type S: an aggregate value cannot contain itself.
* But S may declare a field of the pointer type *S, which lets us create recursive data structures like linked lists and trees.
* The struct type with no fields is called the empty struct, written struct{}
* Some Go programmers use it instead of bool as the value type of a map that represents a set, to emphasize that only the keys are significant

### struct literals

Two forms:

* The first form: a value be specified for every field, in the right order.

```go
type Point struct{ X, Y int}
p := Point{1,2}
```
* The second form: a struct value is initialized by listing some or all of the field names and their corresponding values. Because names
are provided, the order of fields doesn’t matter.
 
```go
anim := gif.GIF{LoopCount: nframes}
```

* The two forms cannot be mixed in the same literal.

* Struct values can be passed as arguments to functions and returned from them (passed by value, only a copy of the data)

```go
func Scale (p Point, factor int) Point {
	return Point{p.X * factor, p.Y * factor}
}

fmt.Println(Scale(Point{1,2},5))
```

of by large structs are usually passed indirect by using a pointer (passed by reference, direct change of the original data)

```go
func Bonus (e *Employee, percent int) int {
	return e.Salary * percent / 100
}
```

* Shorthand to initialize a struct: `pp := &Point{1,2}`

* If all the fields of a struct are comparable, the struct itself is comparable

#### Struct Embedding and Anonymous Fields

```go
type Point struct { X, Y int}
type Circle struct { Center Point, Radius int}
type Wheel struct { Circle Circle, Spokes int}

var w Wheel
w.Circle.Center.X = 8  // or shorthand: w.X 
w.Circle.Center.Y = 8  // or shorthand: w.Y
w.Circle.Radius = 5    // or shorthand: w.Radius = 5
w.Spokes = 20
```

* Go lets us declare a field with a type but no name; such fields are called anonymous fields. -> shorthand possible
* The type of the field must be a named type or a pointer to a named type. 
* Circle and Wheel have one anonymous field each. 
* We say that a Point is embedded within Circle, and a Circle is embedded within Wheel.
* Unfortunately, there’s no corresponding shorthand for the struct literal syntax,
```go
w = Wheel{Circle{Point{8,8},5},20}
w = Wheel{
	Circle: Circle{
		Point: Point{X: 8, Y: 8},
		Radius: 5,
		},
	Spokes: 20, // NOTE: trailing comma necessary here (and at Radius)
	}

w = Wheel{8, 8, 5, 20} // compile error
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // compile error
```

* Because "anonymous" fields do have implicit names, you can’t have two anonymous fields of the same type since their names would conflict.
* In the examples above, the Point and Circle anonymous fields are exported. Had they been unexported (point and circle), we could still use the shorthand form w.X = 8 // equivalent to w.circle.point.X = 8 but the explicit long form shown in the comment would be forbidden outside the declaring package because circle and point would be inaccessible.
* anonymous fields need not be struct types; any named type or pointer to a named type will do. Will also used in methods




## Typgleichheit

```
type Celsius float64
type Fahrenheit float64
```

Beide sind vom Typ float64, da sie aber unterschiedliche Namen haben, sind sie nicht gleich, also Typ Celsius != Typ Fahrenheit
Möchte man die beiden vergleichen, muss eine Typ-Konvertierung durchgeführt werden, also 
 
```
var c Celsius
var f Fahrenheit

c == Celsius(f)   <- true
```


# Scope

Hier wird zuerst cwd außen initialisiert und dann noch einmal im inner Block ein weiteres cwd und err. Wenn der Block verlassen wird, ist cwd nicht gefüllt, da die Variable out of Scope ist. 

```go
var cwd string

func init() {
    cwd, err := os.Getwd() // NOTE: wrong!
    if err != nil {
        log.Fatalf("os.Getwd failed: %v", err)
        }
    log.Printf("Working directory = %s", cwd)
    }

```

Die Intention ist aber eine andere, daher darf man cwd nicht im inneren Block neu definieren. Hier wird nur err definiert, cwd wurde ja bereits außerhalb des Blocks definiert. Dann ist cwd auch gefüllt im äußeren Block.

```go
var cwd string

func init() {
    var err error
    cwd, err = os.Getwd()
    if err != nil {
        log.Fatalf("os.Getwd failed: %v", err)
        }
    }
```