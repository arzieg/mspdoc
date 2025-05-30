# Profiling



Einbinden von pprof in den Code notwendig: 

```go
import (
    _ "net/http/pprof"
    ...
)
```

Aufruf: http::/\<server\>/debug/pprof

## How to profile

1. Build the program: go build .
2. http://localhost:8080/debug/pprof/profile (after 30sec download a profile file)
3. run: go tool pprof <binary> <profile-file> in one of three ways:
    * interactive, with a prompt
    * just get the top entries with -top
    * open a browser with -http=":6060"


## Performance Optimization: pprof

Golang’s concurrency model is powerful, but mismanaged Goroutines can lead to memory leaks and high CPU usage. The pprof package helps diagnose performance bottlenecks by providing real-time profiling data.

Example: Enabling pprof on a go server

```
import _ "net/http/pprof"

go func(){
    log.Println(http.ListenAndServe("localhost:6060", nil))
}()
```

open http://localhost:6060/debug/pprof/ in your browser to inspect CPU, memory, and Goroutine usage in real time.

https://pkg.go.dev/net/http/pprof
