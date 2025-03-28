# Filelayout

* Typdeklarationen, Constanten und Methoden werden in einem go-File abgelegt
* Funktionen werden in einem anderem go-File abgelegt (analog c mit .c und .h?)






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

* The built-in function make creates a slice of a specified element type, length, and capacity. The capacity argument may be omitted, in which case the capacity equals the length.

```go
make([]T, len)
make([]T, len, cap) // same as make([]T, cap)[:len]
```

* Under the hood, make creates an unnamed array variable and returns a slice of it; the array is accessible only through the returned slice. In the first form, the slice is a view of the entire array. In the second, the slice is a view of only the array’s first len elements, but its capacity includes the entire array. The additional elements are set aside for future growth.









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