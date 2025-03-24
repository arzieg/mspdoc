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