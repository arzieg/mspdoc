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

inefficient prime number calculator
```go
package main

import "fmt"

func generator(limit int, ch chan<- int) {
	for i := 2; i < limit; i++ {
		ch <- i
	}
	close(ch)
}

func filter(src <-chan int, dst chan<- int, prime int) {
	for i := range src {
		if i%prime != 0 {
			dst <- i // prime number
		}
	}
	close(dst)
}

func sieve(limit int) {
	ch := make(chan int)

	go generator(limit, ch)

	for {
		prime, ok := <-ch
		if !ok {
			break
		}
		ch1 := make(chan int)
		go filter(ch, ch1, prime)
		ch = ch1
		fmt.Print(prime, " ")
	}
}

func main() {
	sieve(100)
}
```


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
package main

import (
	"log"
	"time"
)

func main() {
	chans := []chan int{
		make(chan int),
		make(chan int),
	}

	for i := range chans {
		go func(i int, ch chan<- int) {
			for {
				time.Sleep(time.Duration(i) * time.Second)
				ch <- i
			}

		}(i+1, chans[i])
	}

	for i := 0; i < 12; i++ {
		select {
		case m0 := <-chans[0]:
			log.Println("received", m0)
		case m1 := <-chans[1]:
			log.Println("received", m1)
		}
	}
}
```

Example: Timeout, you could use 
```go
stopper := time.After(3 * time.Second)  // time.After is a chan and send a signal

...
for range list {
  select {
    case r:= <- result:
      ...
    case <-stopper:
      log.Fatal("timeout") // kill program
  }
...
}
```

**default**

In a select block, the default case is always ready and will be chosen if no other case is.

**Do not use default inside a loop - the select will busy wait and waste CPU**

Book recomendation: Concurrency in go, Katherine Cox-Buday


# Gotchas

Concurrency problems: 
1. race conditions: with -race go find some race conditions
2. deadlock: go find some deadlocks automatically
3. goroutine leak: 
	* goroutine hangs on a empty or blocked channel
	* no daedlock: other goroutines make progress
	* often found by looking at pprof output\
	**when you start a goroutine always know how/when it will end**
4. channel errors:
	* trying to send on a closed channel
	* trying to send or receive on a nil channel
	* closing a nil channel
	* closing a channel twice
5. other errors:
	* closure capture
	* misuse of mutex, WaitGroups, select

(Document: https://cseweb.ucsd.edu/~yiying/GoStudy-ASPLOS19.pdf)

Mutex:
 Mutexe müssen in einer Reihenfolge aufgebaut werden und in der umgekehrten Reihenfolge wieder gelöscht werden. 

Bsp. Goroutine Leak, Solution: put buffer to ch, so it could not block
```go
func finishReq(timeout time.Duration) *obj{
	ch := make(chan obj)
	go func() {
		.. // work that takes to long
		ch <- fn()  // blocking send
	}()

	select {
		case rslt := <-ch
			return rslt
		case <- time.After(timeout):
			return nil
	}
}
```
	
WaitGroups: 
	always Add (wg.Add(1)) before unit of work starts (unmittelbar dovor)

Bsp.: closure capture. 
A goroutine closure shouldn't capture a **mutating** variable

```go
for i :=0; i<10; i++ { // Wrong
	go func() {
		fmt.Println(i)
	}()
}

for i :=0; i<10; i++ { // right
	go func() {
		fmt.Println(i)
	}(i)   // pass the variable values as a parameter
}
```

**select problems:**
* default is always active
* a nil channel is always ignored
* a full channel (for send) is skipped over
* a "done" channel is just another channel
* available channels are selected at random


