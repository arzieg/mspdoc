# Benchmark

Go has standard tools and conventions for running benchmarks

Benchmarks live in test files ending with _test.go
Run benchmark with: go test -bench=. -benchtime=4s -benchmem -cpu=2,4,8 ./fib/main_test.go, 

Go only runs the BenchmarkXXX functions, (benchtime als Parameter kann die Laufzeit bestimmt werden, benchmem=ein paar mehr Statistiken, -cpu=#Anzahl der CPU)

```go
func BenchmarkFib20T(b *testing.B) {
    ...

    b.ResetTimer()  <- bestimmen, ab wann man messen möchte
}
```

Calling lots short methods via dynamic dispatch is very expensive
The cost of calling a function should be proportional to the work it does (short inline functions vs. longer methods with late binding)


## Inline directive

Man kan inline directiven setzen, um z.B. Optimierungen durch den Compiler zu umgehen. 

```go
//go:noinline 
```

## A few things to consider

Here are some concerns about CPU/memory benchmarking

* is the data / code available in cache? 
* did you hit a garbage collection
* did virtual memory have to page in/out
* did branch prediction work the same way? 
* did the compiler remove code via optimization? (are there side effects in the code?)
* are you running in parallel? How many cores? 
* are those cores physical or virtual? 
* are you sharing a core with anything else? 
* whar other processes are sharing the machine? 


