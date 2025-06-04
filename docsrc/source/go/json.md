# JSON

https://go.dev/blog/json

## generic json

Wenn man die JSON Struktur nicht kennt, kann die Datenstruktur über ein generisches JSON gelesen werden. 
Hierzu definiert man ein leeres Interface{}, der als generischer Container fungiert. 
Über type assertion wird daraus dann ein konkreter Typ. 

```go
var i interface{}
i = "a string"
i = 2011
i = 2.777

r := i.(float 64)   // type assertion 

Wenn der Typ unbekannt ist, kann auch über Switch ermittelt werden. 

switch v := i.(type) {
case int:
    fmt.Println("twice i is", v*2)
case float64:
    fmt.Println("the reciprocal of i is", 1/v)
case string:
    h := len(v) / 2
    fmt.Println("i swapped by halves is", v[h:]+v[:h])
default:
    // i isn't one of the types above
}
```

Das **json** Package nutzt map[string]interface{} und []interface{} Werte um das JSON Object zu speichern. Daher kann Interface{} auch verwendet werden, wenn man die JSON Struktur nicht kennt. 

### Decoding generic json

Bsp.: b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)

Go speichert dies in einer generischen Map mit leeren Interface: 
```go
f = map[string]interface{}{
    "Name": "Wednesday",
    "Age":  6,
    "Parents": []interface{}{
        "Gomez",
        "Morticia",
    },
}
```

Um auf die Daten zuzugreifen, muss eine type assertion erfolgen 

`m := f.(map[string]interface{})`

Über das kann dann iteriert werden. 

```go
for k, v := range m {
    switch vv := v.(type) {
    case string:
        fmt.Println(k, "is string", vv)
    case float64:
        fmt.Println(k, "is float64", vv)
    case []interface{}:
        fmt.Println(k, "is an array:")
        for i, u := range vv {
            fmt.Println(i, u)
        }
    default:
        fmt.Println(k, "is of a type I don't know how to handle")
    }
}
```

### Streaming Encoders und Decoders

NewDecoder und NewEncoder Funktionen wrapen io.Reader und io.Writer

```go
func NewDecoder(r io.Reader) *Decoder
func NewEncoder(r io.Writer) *Encoder
```

Bsp. lesen von JSON Objekten von Standard Input und entfernen des Keys Name, dann Schreiben nach StdOut. 

```go
package main

import (
    "encoding/json"
    "log"
    "os"
)

func main() {
    dec := json.NewDecoder(os.Stdin)
    enc := json.NewEncoder(os.Stdout)
    for {
        var v map[string]interface{}
        if err := dec.Decode(&v); err != nil {
            log.Println(err)
            return
        }
        for k := range v {
            if k != "Name" {
                delete(v, k)
            }
        }
        if err := enc.Encode(&v); err != nil {
            log.Println(err)
        }
    }
}
```





