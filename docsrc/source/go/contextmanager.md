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


