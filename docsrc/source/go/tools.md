# Tools

## Local Development Environment: ServBay

When to Use ServBay?

* Need to quickly set up a Golang + MySQL/PostgreSQL/Redis environment on your local machine
* Work on multiple projects spanning different stacks (Golang + PHP + Node.js) and require easy version switching
* Need HTTPS/SSL certificates but don’t want to manually configure Nginx

ServBay eliminates tedious manual setup, letting you focus on coding rather than environment issues.
https://www.servbay.com/

## Code Quality Checking: golangci-lint

https://golangci-lint.run/

## Dependency Management: Go Modules

https://go.dev/ref/mod

## Test Automation: Gotests

https://github.com/cweill/gotests

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

