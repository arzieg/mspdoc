# functions

Quelle: The GO programming language, Alan A.A. Donovan, Brian W. Kernighan

## first class functions

In go a function is a first class function, that means

* define them - even inside an other function
* create anonymous function literals
* pass them as function parameters / return values
* store them as variables
* store them in slices and maps (but no keys)
* store them as fields of a structure type
* send and receive them in channels
* write methods against a function type
* compare a function var agains nil

## function scope

alles was man in packages dekaliert kann man auch in Funktionen deklarieren

eine Funktion ist gleich, wenn sie die gleiche Signatur hat (also gleiche Typen für aufrufende und rückgebende Parameter als auch gleiche Reihenfolge):

```
var try func(string, int) string
func Do(a string, b int) string { ... }

func NotDo(x string, y int) (a string) {...}
```

a parameter is passed by value if the function gets a copy
a parameter is passed by reference (technically by value but it works like a reference) if the function can modify the actual parameter

Funktionen können mehr als ein Rückgabewert haben
func doIt( a int, b []int)(int, error)

Funktionen unterstützen Rekursion


## by value / by reference

By value: 
* numbers
* bool
* arrays
* structs

By reference:
* passed by pointers (&x)
* strings (but they are immutable)
* slices
* maps
* channels

Beispiel map: 

```
package main

import "fmt"

func do(m1 *map[int]int) {
	(*m1)[3] = 0
	(*m1) = make(map[int]int)  <- descriptor wird überschrieben von der main function m=m1, also eigentlich wird neues Object erstellt und der descriptior m wird ebenfalls geändert (dies Daten nicht)
	(*m1)[4] = 4
	fmt.Println("m1", *m1)
}

func main() {
	m := map[int]int{4: 1, 7: 2, 8: 3}
	fmt.Println("m", m)
	do(&m)   <- call bei reference
	fmt.Println("m", m)

}

Ausgabe:
m map[4:1 7:2 8:3]
m1 map[4:4]
m map[4:4]

```

## Defer

defer - mache noch etwas, wenn die function beendet worden ist

* close a file we openend
* close a socket
* unlock a mutex we locked
* save something before we done
...

defer arbeitet im Scope einer Funktion (nicht auf Block Ebene)
Parameter in defer werden kopiert, d.h. je nachdem wo defer definiert wurde, werden diese Werte kopiert zu diesem Zeitpunkt

defer sollte man erst dann definieren, wenn man vorher gerpüft hat, das ein Zustand auch existiert, bsp.: 
Erst prüfen, ob eine Datei geöffnet werden kann. Wenn das der Fall ist, dann defer f.Close() definieren, ansonsten kann es passieren, dass defer auf eine nicht vorhandene Datei angewandt wird und ein Fehler geworfen wird.

```go
func main() {
  for i :=1; i<len(os.Args); i++ {
    f,err := os.Open(os.Args[i])
    ...
    defer f.Close()    <- defer schliesst jede einzelne Datei (also je File wird ein Defer definiert)
    ...
  }
} <- defer greift erst nach verlassen der funktion, d.h. erst hier werden die Dateien geschlossen (also Programmlogik beachten, defer nicht immer die beste Wahl)
In diesem Fall, besser f.Close() direkt aufrufen, nachdem man mit der Datei gerarbeitet hat.
```


defer kann nur auf Funktionen angewendet werden. Wenn man einen Wert einer bspw. int Variable ändern möchte, muss man das über anonyme functions machen:
(sollte man nicht unbedingt, da doch sehr verwirrend)

```go

func doIt() (a int){
    defer func() {
        a = 2
    }()   <- anoynme function 
    a=1
    return <- return wert jetzt 2 
}
```

## anonyme funktionen

a function literal to denote a function value within any expression.

TODO: ausarbeiten


## variadic functions

A variadic function is one that can be called with varying numbers of arguments. The most familiar examples are fmt.Printf and its variants.

Declaration: ...int


## defer

Syntactically, a defer statement is an ordinary function or method call prefixed by the keyword defer. The function and argument expressions are evaluated when the statement is executed, but the actual call is deferred until the function that contains the defer statement has finished, whether normally, by executing a return statement or falling off the end, or abnormally, by panicking.

Any number of calls may be deferred; they are executed in the reverse of the order in which they were deferred.

The defer statement can also be used to pair "on entry" and "on exit" actions when debugging a complex function.

```go
func bigSlowOperation() {
	defer trace("bigSlowOperation")() // dont forget the extra parentheses
	time.Sleep(10 * time.Second)      // simulate slow operation
}

func trace(msg string) func() {
	start := time.Now()
	log.Printf("enter %s", msg)
	return func() { log.Printf("exit %s (%s)", msg, time.Since(start)) } // return anonymous function
}

func main() {
	bigSlowOperation()
}
```

Deferred functions run after return statements have updated the function’s result variables. Because an anonymous function can access its enclosing function’s variables, including named results, a deferred anonymous function can observe the function’s results.

Because deferred functions aren’t executed until the very end of a function’s execution, a defer statement in a loop deserves extra scrutiny.

Wrong: 

```go
for _, filename := range filenames {
  f, err := os.Open(filename)
  if err != nil {
    return err
  }
  defer f.Close() // NOTE: risky; could run out of file descriptors
// ...process f...
}
```

Better:

```go
for _, filename := range filenames {
  if err := doFile(filename); err != nil {
  return err
  }
}

func doFile(filename string) error {
  f, err := os.Open(filename)
  if err != nil {
  return err
  }
  defer f.Close()
// ...process f...
}
```


