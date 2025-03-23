# Filelayout

* Typdeklarationen, Constanten und Methoden werden in einem go-File abgelegt
* Funktionen werden in einem anderem go-File abgelegt (analog c mit .c und .h?)






# Types

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


