# Printf

# Abkürzungen

https://cheatography.com/fenistil/cheat-sheets/go-fmt-formattings/


Default formats and type

| format  | %  |
|---------|----|
|Default  | %v |
|Golang Syntax | %#v |
|Type          |  %T |



## Wiederholung

Mit [x] kann man den Parameter x wiederholen und muss nur einmal den Parameter bei Printf angeben. Bsp.: mit [1] wird der erste Parameter
wiederholt (also hier o)

```go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
// Output:
// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

