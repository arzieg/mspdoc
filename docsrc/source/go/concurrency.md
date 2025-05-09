# Go routines and channels

Go enables two styles of concurrent programming. 
* goroutines and channels, which support communicating sequential processes or CSP, a model of concurrency in which values are passed between independent activities (goroutines) but variables are for the most part confined to a single activity. 

* a more traditional model of shared memory multithreading, which will be familiar if you’ve used threads in other mainstream languages. 

# Concurrency

Concurrency := Parts of the program may execure independently in some non-deterministic (partial) order 

* You can have concurrency with a single core processor - think interrupt handling in the os
* Concurrency is about dealing with things happening out-of-order

Parallelism := Parts of a program execute independently at the same time 
* parallelism can happen only on a multi-core processor
* Parallelism is about things actually happening at the same time
* We need concurrency to allow parts of the program to execute independently

Concurrency doesn't make the program faster, parallelism does. 
A single program won't have parallelism without concurrency

Race condition := System behavior depends on the (non-deterministic) sequence of timinig of parts of the program executing independently, where some possible behaviors (order of execurtion) produce invalid results

How to solve:
* don't share anything
* make the shared things read-only
* allow only one write to the shared things
* make the read-modify-write operations atomic (we are adding more (sequentual) order to our operations)

## Channels /Pipes

* things go in one end, come out the other
* in the same order they went in
* unti the channel is closed
* multiple readers & writers can share it safely

Each sequential process can run independently and can use channels

CSP (Communicating sequential processes) provides a model of thinking about it that makes it less hat 

A Channel is a vehicle for transfering ownership of data, so that only on goroutine at a time is writing the data (avoid race conditions)

"Don't communicate by sharing memory; instead, share memory by communicating" - Rob Pike

* A channel (as map) is a reference to the data structure created by make.
* Copy a channel or pass as an argument to a function, we are copying a reference, so caller and callee refer to the same data structure.
* Zero Value of a channel is NIL
* Two Channels of the same type may be compared using ==
* A channel may also be compared to nil.
* close a channel: close(ch)

ch <- x   // a send statement
x = <-ch  // a receive expression in an assignment statement
<-ch      // a receive statement; result is discarded

ch = make(chan int)   // unbuffered channel
ch = make(chan int,0) // unbuffered channel
ch = make(chan int,3) // buffered channel with capacity 3

### unbuffered channels
* a send operation on an unbuffered channel blocks the sending goroutine
* Communication over an unbuffered channel causes the sending and receiving goroutines to synchronize.
* When a value is sent on an unbuffered channel, the receipt of the value happens before the reawakening of the sending goroutine.



## Goroutine
goroutine is a unit of independent execution (coroutine), put go for function call
How to stop:
* you have a well defined loop terminating condition
* you signal completion through a channel or context
* you let it run until the program stops
-> but you need to make sure it doesn't get blocked by mistake


## Pipelines

Channels can be used to connect goroutines together so that the output of one is the input to another. This is called a pipeline.

There is no way to test directly whether a channel has been closed, but there is a variant of the receive operation that produces two results: the received channel element, plus a boolean value, conventionally called ok, which is true for a successful receive and false for a receive on a closed and drained channel.

You needn’t close every channel when you’ve finished with it. It’s only necessary to close a channel when it is important to tell the receiving goroutines that all data have been sent

## select

select allows any ready alternative to proceed among
* a channel we can read from
* a channel we can write to
* a default action that's always ready

Mosten often *select* runs in a loop se we keep trying

We can put a timeout or "done" channel into the *select*

* we can compose channels as sysnchronization primitives
* traditional primitives (mutex, condition variable) can't be composed

```go
for i:= 0; i<12; i++ {
    select {
        case m0:= <-chans[0]:
          log.Println("received", m0)
        case m1:= <-chans[1]:
          log.Println("received", m1)
    }
}