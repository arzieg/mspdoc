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

## Goroutine
goroutine is a unit of independent execution (coroutine), put go for function call
How to stop:
* you have a well defined loop terminating condition
* you signal completion through a channel or context
* you let it run until the program stops
-> but you need to make sure it doesn't get blocked by mistake


