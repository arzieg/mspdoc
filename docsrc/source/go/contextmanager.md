# Contextmanager

## Cancelation and timeouts

The context package offers a common method to cancel requests
* explicit cancellation
* implicit cancellation based on a timeout or deadline

A context may also carry requests-specifig values, such as a traceID

A cointext offers two controls: 
* a channel that closes wheen the cancellation occurs
* an error that is readable once the channel closes

The error value tells you whether the requests was cancelled or timed out

We often use the channel from Done() in a select block

Context from an **immutable** tree structure (goroutine-safe; changes to a context do not affect ist ancestors)

Cancellation or timeout applies to the current context and its **subtree*

A subtree may be created with a shorter timeout (but not longer)

**Context is a tree of immutable nodes which can be extended**

```go
ctx := context.Background()
ctx = context.WithValue(ctx, "v",7)
ctx, canc := context.WithTimeout(ctx,t)
req, _ := http.NewRequest(method, url, nil)
req = req.WithContext(ctx)
resp, err := http.DefaultClient.Do(req)
```

The Context value should ALWAYS be the first parameter (convention)

Example:
```go
package main

import (
	"context"
	"log"
	"net/http"
	"time"
	//"golang.org/x/net/context"
)

type result struct {
	url     string
	err     error
	latency time.Duration
}

func get(ctx context.Context, url string, ch chan<- result) {
	start := time.Now()
	req, _ := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if resp, err := http.DefaultClient.Do(req); err != nil {
		ch <- result{url, err, 0}

	} else {
		t := time.Since(start).Round(time.Microsecond)
		ch <- result{url, nil, t}
		resp.Body.Close()
	}
}
func main() {
	results := make(chan result)
	list := []string{
		"https://amazon.com",
		"https://google.com",
		"https://nytimes.com",
		"https://wsj.com",
		"http://localhost:8000/slow",
	}

	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)

	defer cancel()

	for _, url := range list {
		go get(ctx, url, results)
	}

	for range list {
		r := <-results

		if r.err != nil {
			log.Printf("%-20s %s\n", r.url, r.err)
		} else {
			log.Printf("%-20s %s\n", r.url, r.latency)

		}
	}

}
```

In the following example only the first responder should be printed. The trick is her to make a channel and limit the capacity to the max. urls of the list:

```go
package main

import (
	"context"
	"log"
	"net/http"
	"runtime"
	"time"
)

type result struct {
	url     string
	err     error
	latency time.Duration
}

func get(ctx context.Context, url string, ch chan<- result) {

	var r result

	start := time.Now()
	ticker := time.NewTicker(1 * time.Second).C
	req, _ := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)

	if resp, err := http.DefaultClient.Do(req); err != nil {
		r = result{url, err, 0}
	} else {
		t := time.Since(start).Round(time.Microsecond)
		r = result{url, nil, t}
		resp.Body.Close()
	}

	for {
		select {
		case ch <- r:
			return
		case <-ticker:
			log.Println("tick", r)
		}
	}
}

func first(ctx context.Context, urls []string) (*result, error) {
	// es kann sein, dass sender sendet, aber receiver noch nichts empfängt, weil noch nicht gestartet.
	// das kann zum Hochlaufen führen. mit len(urls) kann man die Kapazität begrenzen
	results := make(chan result, len(urls)) // buffer to avoid leaking
	ctx, cancel := context.WithCancel(ctx)

	defer cancel()

	for _, url := range urls {
		go get(ctx, url, results)

	}
	select {
	case r := <-results: // when return also defer cancel() will run
		return &r, nil
	case <-ctx.Done(): // handle timeout from context above, if that happens. I have to handle this
		return nil, ctx.Err()
	}
}

func main() {

	list := []string{
		"https://amazon.com",
		"https://google.com",
		"https://nytimes.com",
		"https://wsj.com",
		"http://localhost:8000/slow",
	}

	r, _ := first(context.Background(), list)

	if r.err != nil {
		log.Printf("%-20s %s\n", r.url, r.err)
	} else {
		log.Printf("%-20s %s\n", r.url, r.latency)

	}

	time.Sleep(9 * time.Second)
	log.Println("quit anyway...", runtime.NumGoroutine(), "still running")

}
```



Context values should be data specific to a request, such as:
* a traceID or start time (for latency calculation)
* security or authorization data

**Avoid** using the context to carry "optional" parameters

Use a package-specifix, private context key type (not string) to avoid collisions
```go
type contextKey int

const TraceKey contextKey=1  // export parameter

// AddTrace is HTTP middleware to insert a trace ID into the request
func AddTrace(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request){
		ctx := r.Context()
		if traceID := r.Head.Get("X-Cloud-Trace-Context"); traceID != "" {
			ctx = context.WithValue(ctx, TraceKey, traceID)
		}
		next.ServerHTTP(w, r.WithContext(ctx))
	})
}
```


