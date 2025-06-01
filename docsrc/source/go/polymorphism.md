# Parametric Polymorphism

## Generics in Go

Generics is shorthand for parametric polymorphism

That means we have a type parameter on a type or function

```go
type MyType[T any] struct {
    v T   // can be any valid Go type
    n int
}
```

**Generics are a powerfil feature for abstraction. And a possible source of unnecessary abstraction and complexity.**

## When to use

Use type parameters to **replace dynamic typing** with static typing

example: 
 interface{} + v(T) //Downcast type -> in generics: type MyType[T any] struct { ... }

If it runs faster, consider that a bonus. Continue to use (non empty) interfaces whereever possible. Performance should not be you principal reason for generics (in most cases)

## Generic type and function

```go
type Vector[T any] []T   // any ist ein alias für ein leeres interface{}

func (v *Vector[T]) Push(x T) { // hier wir ein slice geändert, da *Vector wichtig, ansonsten wäre es eine Kopie
  *v = append(*v, x)  // may reallocate
}

// note: F and T are both used in the parameter list
// Map[...](allg. Parameter für func). [] ist neu, in anderen Sprache ist das < >
func Map[F, T any](s []F, f func(F) T) []T{  // 2 Typ parameter F,T, funktion mit zwei Übergabeparametern: s(lice) []F und eine function von Typ F und konvertiert das nach T) , Rückgabe ein Slice vom Typ []T
    r := make([]T, len(s))

    for i,v := range s {
        r[i] = f(v)
    }
    return r
}

func main() {
    s:= Vector[int]{}  // 
    s.Push(1)
    s.Push(2)

    t1 := Map(s, strconv.Itoa)  // get i convert to str - it works becaus Itoa produce no err 
    // man beachte [F, T any] wird gar nicht mit aufgerufen also bspw. Map[int,string](...)
    // der compiler löst das auf, da genug informationen vorliegen, da sowohl F und T in (...) 
    // als Inputparameter verwendet werden.
    t2 := Map([]int{1,2,3}, strconv.Itoa)

    fmt.Println(s)  // [1 2]  
    fmt.Printf("%#v\n", t1) // []string{"1","2"}
    fmt.Printf("%#v\n", t2) // []string{"1","2","3"}

}
```

```go
type num int

func (n num) String() string {
    return strconv.Itoa(int(n))  // itoa erwartet ein int, n ist vom typ num, daher typecast notwendig  
}

type StringableVector[T fmt.Stringer] []T

func (s StringableVector[T]) String() string {
    var sb strings.Builder

    sb.WriteString("<<")

    for i,v := range s{  // s is a slice of type T, v is a Element for Type T
        for i > 0 {
            sb.WriteString(", ")
        }
        sb.WriteString(v.String())  // v has a String method
    }
    sb.WriteString(">>")
    return sb.String()
}

func main(){
    var s StringableVector = []num{1,2,3}  // hier compiler fehler
    // cannot use generic type StringavleVector[T fmt.Stringer] without instantiation
    // []num{1,2,3} ist für den compiler nicht ausreichend um festzustellen, das es sich um ein StringavleVector[num] handelt, daher
    var s StringableVector[num] = []num{1,2,3}
    fmt.Println(s) // <<1,2,3>>

}
```





