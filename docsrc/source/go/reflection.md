# Reflection

## type assertion

"interface{}" says nothing since it has no methods

It is a generic thing, but sometimes we need its real type

We can extract a specific type with a type assertion (aka downcasting)

This has the form value.(T) for some type T

```go
var w io.Writer = os.Stdout

f:= w.(*os.File)  // success: f==os.Stdout
c:= w.(*bytes.Buffer) // panic: interface holds *os.File, not *bytes.Buffer
```

If we use the two-result version, we can avoid panic:

```go
var w io.Writer = os.Stdout

f,ok := w.(*os.File)  // success: ok, f==os.Stdout
b, ok:= w.(*bytes.Buffer) // failure: !ok, b==nil
```

## Deep equality

We can use the reflect package in UTs to check equality

```go
want := struct{
    a: "a string",
    b: []int{1,2,3}  // not comparable with == 
}

got := gotGetIt(...)

if !reflect.DeepEqual(got, want) {
    t.Errorf("bas response: got=%#v, want=%#v", got, want)
}
```

github.com/kylelemon/godebug/pretty show a deep diff

We can also use type assertion in a switch statement (matching a type not a value)

```go
func Println(args ...interface{}){
    ...
    for arg := range args {
        switch a := arg.(type) {
            case string: ...    // concrete type
            case Stringer: ...  // interface
        }
    }
}
```

Here the switch variable **a** has a specific type if the case has a *single* type

