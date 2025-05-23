# Error 


The fmt.Errorf function formats an error message using fmt.Sprintf and returns a new error value. We use it to **build descriptive errors** by successively prefixing additional context information to the original error message. When the error is ultimately handled by the program’s main function, it should provide a clear causal chain from the root problem to the overall failure, reminiscent of a NASA accident investigation:

*genesis: crashed: no parachute: G-switch failed: bad relay orientation*

Because error messages are frequently chained together, message strings **should not be capitalized** and **newlines should be avoided**. The resulting errors may be long, but they will be self-contained when found by tools like grep.

```go
doc, err := html.Parse(resp.Body)
    resp.Body.Close()
    if err != nil {
        return nil, fmt.Errorf("parsing %s as HTML: %v", url, err)   <-- Errorf, Error wird angereichert mit einer weiteren Meldung
    }
```

## mit Timeout

```go
// WaitForServer attempts to contact the server of a URL.
// It tries for one minute using exponential back-off.
// It reports an error if all attempts fail.
func WaitForServer(url string) error {
    const timeout = 1 * time.Minute
    deadline := time.Now().Add(timeout)
    for tries := 0; time.Now().Before(deadline); tries++ {
        _, err := http.Head(url)
        if err == nil {
            return nil // success
        }
    log.Printf("server not responding (%s); retrying...", err)
    time.Sleep(time.Second << uint(tries)) // exponential back-off
    }
    return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}
```

## Ende vom Prozess

Third, if progress is impossible, the caller can print the error and stop the program gracefully, but this course of action should generally be reserved for the main package of a program. Library functions **should usually propagate errors to the caller**, unless the error is a sign of an
internal inconsistency—that is, a bug.

```go
// (In function main.)
if err := WaitForServer(url); err != nil {
    fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)    <-- hier wird Fprintf genutzt
    os.Exit(1)
}
```

## oder logging

### Fatalf

A more convenient way to achieve the same effect is to call **log.Fatalf**. As with all the log functions, by default it prefixes the time and date to the error message.

```go
if err := WaitForServer(url); err != nil {
    log.Fatalf("Site is down: %v\n", err)
}
```

When you call log.Fatalf, the program doesn't just exit; it drops the misc, walks off stage, and doesn’t care what’s still running—files, goroutines, you name it. This can lead to problems if cleanup is needed.

Use log.Fatalf when:
* The program cannot logically continue. Think missing configurations, critical services being unavailable, or a sanity check failing.
* Recovery is pointless. Retrying or fallback logic isn’t viable in this context.
* It’s during setup or initialization. If your app can’t even get started properly, just fail fast and spare everyone the pain.

### Errorf

Use log.Errorf when:
* The error is important but not fatal. Log it, maybe recover, or gracefully degrade functionality.
* You need logs to debug the issue later, but the app can limp along in the meantime.
* The issue is temporary. Network problems, for example, might resolve with retries.

```go
package main
import (
	"log/slog"
	"os"
)
func main() {
	logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
	logger.Info("User logged in", "userID", 123)


    if err := sendEmail(); err != nil {
        logger.Errorf("Failed to send email: %v", err)
        // Maybe retry or continue with other operations
    }
}
```


## Custom error type

```go
type errKind int

const (
    -   errKind = iota
    noHeader
    cantReadHeader
    ...
)

type WaveError struct {
    kind errKind
    value int
    err error
}

// Implement a error method
func (e WaveError) Error() string {
    swith e.kind {
        case noHeader:
            return "no header (file to short?)"
        case cantReadHeader:
            return fmt.Sprintf("can't read header[%d]: %s", e.value, e.err.Error())
        ...
    }
}

// some helper methods
// with returns an error with a particular value (e.g. header type)
func (e WaveError) with(val int) WaveError {
    e1 := e
    e1.value = val
    return e1
}

// from returns an error with a particular location and underlying error (e.g. from the standard library)
func (e WaveError) from (pos int, err error) WaveError {
    e1 := e
    e1.value = pos
    e1.err = err
    return e1
}

// some prototypes, which are exported
var (
    HeaderMissing = WaveError{kind: noHeader}
    HeaderReadFailed = WaveError{kind: cantReadHeader}
    ...
)

// and in use
func DecodeHeader(b []byte) (*Header, []byte, error){
    ...
    if len(b) < headerSize {
        return &header, nil, HeaderMissing
    }
}

```

In go >= 1.13 error could be wraped (%w format)

```go
...
  return fmt.Errorf("Im sorry %s, I cant't: %w", h.victim, h.err)
...
```
=> Error chaing: top-level-error -> intermediate error -> original error

Custom error types may now unwrap their internal errors

```go
func (w *WaveError) Unwrap() error {
    return w.err
}
```

## Error.Is (compares an error with a variable)

we can check whether an errar has another error in its chain

errors.Is compares with an error **variable**, not a type

ex.  if errors.Is(err, os.ErrPermission) ...

## Error.As (compares an error with a type)

errors.As looks for an error type, not a value

ex.: 
```go
...
var e os.PathError

if errors.As(err, &e) {
    ....
}
```

