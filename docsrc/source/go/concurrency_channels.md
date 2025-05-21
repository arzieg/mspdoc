# Channels

Channels **block* unless ready to read or write

A channel is ready to write if
* it has buffer space, or
* at least one reader is ready to read (rendezvous)

A channel is ready to read if
* it has unread data in its buffer, or
* at least one writer is ready to write (rendezvous), or
* it is closed


Channels are unidirectional, but have two ends (which can be passed separately as parameters)

```go
func get(url string, ch chan <- result) {  // write only
    ...
}

func collect(ch <-chan result) map[string]int { // read only
    ...
}
```

Closing a channel causes it to return the "zero" value. 

```go
func main(){
    ch := make(chan int, 1)

    ch <- 1

    b, ok := <-ch
    fmt.Println(b, ok) // 1 true

    close(ch)

    c, ok := <-ch
    fmt.Println(c,ok) // 0 false
       
    }
```

If the channel has no buffer, then the above program crash (deadlock)

A channel can only be closed once (else it will panic)

One of the main issues in working with goroutines is **ending** them
* an unbuffered channel requires a reader and writer (a weiter blocked on a channel with no reader will "leak")
* closing a channel is often a **signal** that work is done
* only **one** goroutine can close a channel (not many)
* we may need some way to coordinate closing a channel or stopping goroutines (beyond the channel itself)

## Nil Channels

Reading or writing a channel that is nil always blocks

**but* a nil channel in a *select* block is *ignored*

This can be a powerful tool:
* use a channel to get input
* suspent it by changing the channel variable to nil
* you can even un-suspend it again
* but **close** the channel if there really is no more input (EOF)

## channel state reference

|state      | receive     | send      | close              |
| --------- | ----------- | --------- | ------------------ |
|nil        | block*      | block*    | panic              |
|empty      | block       | write     | close              |
|partly full|read         | write     | readable until empty|
|full       | read        | block     | readable until empty|
|closes     | def. value**|panic      | panic              |
|receive only|ok          |compile error| compile error    |
|send only  |compile error| ok        | ok                 |

\* select ignores a nil channel since it would always block \
\*\* Reading a closed channel returns (\<default-value\>, !ok)


## Rendezvous (like chat)

By default, channels are unbuffered (rendezvous model)

* the sender blocks until the receiver is ready (and vice versa)
* the send alway happens before the receive
* the receive always returns before the send
* **the sender & receiver are synchronized**

## Bufferung (like email post)

Buffering allows the sender to send without waiting

* the sender deposits its item and returns immediately
* the sender blocks only if the buffer is full
* the receiver blocks only if the buffer is empty
* **the sender & receiver run independently**

Buffering allows the sender to send without waiting

Common uses of buffered channels:
* avoid goroutine leaks (from an abandoned channel)
* avoid rendezvous pauses (performance improvement)

Dont buffer until it's needed: *buffering may hide a race condition*

Some testing may be required to find the right number of slots!

Special uses of buffered channels:
* counting semaphore pattern

## counting semaphores

A **counting semaphore** limits work in progress (or occupancy)

Once it is full only one unit of work can enter for each one that leaves. 

We model this with a buffered channel:
* attempt to send(write) before starting work
* the send will block if the buffer is full (occupancy is at max)
* receive (read) when the work is done to free up a space in the buffer (this allows the next worker to start)

## Amdahls law of parallelism

Amdahls law: speedup is limited by the part (not) parallelized

S = 1 / (1 -p + (p/s))

S = Speedup, s = #Processors, p = %the program use parallism


# Conventional Synchronization

Package sync: 
* mutex
* once
* pool
* RWMutex
* WaitGroup

Package sync/atomic for atomic scalar reads and writes

## why

What if multiple goroutines must read and write some data? 

We must make sure only **one** of them can do so at any instant (in the so-called "critical section")

We accomplish this with some type of lock:
* acquire the lock before accessing the data
* any other goroutine will **block** waiting to get the lock
* realease the lock when done


Mutex in action:
```go
type SafeMap struct {
    sync.Mutex  // not safe to copy
    m map[string]int
}

// pointer needed
func (s *SafeMap) Incr(key string){
    s.Lock()
    defer s.Unlock()

    // only one goroutine can execure this code at the same time
    s.m[key]++
}
```

RWMutexes in Action: Sometimes we need to prefer readers to infrequent writers
```go
type InfoClient struct {
    mu      sync.RWMutex
    token   string
    tokenTime time.Time
    TTL     time.Duration
}

// only read mutex needed
func (i *InfoClient) CheckToken() (string, time.Duration) {
    i.mu.RLock()
    defer i.mu.RUnlock()

    return i.token, i.TTL - time.Since(i.tokenTime)
}

// sometimes rw mutex needed
func (i *InfoClient) ReplaceToken(ctx context.Context)(string, error) {
    token, ttl, err := i.getAccessToken(ctx)
    if err != nil {
        return "", err
    }
    i.mu.Lock()
    defer i.mu.Unlock()

    i.token = token
    i.tokenTime = time.Now()
    i.TTL = time.Duration(ttl) * time.Second
    return token, nil
}

```
## only-once execution

A sync.Once object allows us to ensure a function runs only once (only the first call to Do will call the function passed in)

```go
var once sync.Once
var x *singleton

func initialize(){
    x = NewSingleton()
}

func handle(w http.ResponseWriter, r *http.Request){
    once.Do(initialize)
}
```

checking x==nil in the handler is **unsafe**

## Pool

a pool provides for efficient and safe reuse of objects, but it is a container of interface{}

```go
var bufPool = syncPool {
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func Log(w io.Writer, key, val string){
    b := bufPool.Get().(*bytes.Buffer)
    b.Reset()
    // write to it
    w.Write(b.Bytes())
    bufPool.Put(b)
}
```

Other primitives:
* condition variable
* Map (safe containter; uses interfac{})
* WaitGroup


