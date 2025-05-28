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

