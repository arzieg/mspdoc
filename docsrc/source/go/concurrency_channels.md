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
