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