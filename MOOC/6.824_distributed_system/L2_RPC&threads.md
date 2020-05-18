# Lecture 2: Infrastructure: RPC and threads
##### Why Go?
- good support for threads
- convenient RPC
- type safe
- garbage-collected (no use after freeing problems)
- threads + GC is particularly attractive! (找到最后一个使用变量的线程很繁琐)
- relatively simple

After the tutorial, use https://golang.org/doc/effective_go.html

## Threads
- threads are main tool to manager currency.
- a useful structuring tool, but can be tricky
- Go calls them goroutines; everyone else calls them threads

##### Thread = "thread of execution"
- threads allow one program to do many things at once
- each thread executes serially, just like an ordinary non-threaded program
- the threads share memory
- each thread includes some per-thread state:
    - program counter, registers, stack

##### Why threads?
- They express concurrency, which you need in distributed systems
  - I/O concurrency
    - Client sends requests to many servers in parallel and waits for replies.
    - Server processes multiple client requests; each request may block.
    - While waiting for the disk to read data for client X, process a request from client Y.
  - Parallelism
    - Multicore performance
    - Execute code in parallel on several cores.
  - Convenience
    - In background, once per second, check whether each worker is still alive.

##### Alternative to threads -- Event-driven 
Event-driven Or asynchronous program
- write code that explicitly interleaves activities, in a single thread.
- Keep a table of state about each activity, e.g. each client request.
- One "event" loop that:
    - checks for new input for each activity (e.g. arrival of reply from server),
    - does the next step for each activity,
    - updates state.
- Event-driven gets you I/O concurrency,
    - and eliminates thread costs (which can be substantial),
    - but doesn't get multi-core speedup,
    - and is painful to program.
    
##### Threading challenges:
- shared data 
    - race condition
        - use locks (Go's `sync.Mutex`)
        - or avoid sharing mutable data
- coordination between threads
    - e.g. one thread is producing data, another thread is consuming it
        - how can the consumer wait (and release the CPU)?
        - how can the producer wake up the consumer?
    - use Go channels or `sync.Cond` or `WaitGroup`
- deadlock
    - cycles via locks and/or communication (e.g. RPC or Go channels)
## crawler example

##### What is a web crawler?
- goal is to fetch all web pages, e.g. to feed to an indexer
- web pages and links form a graph
- multiple links to some pages
- graph has cycles

##### Crawler challenges
- Exploit I/O concurrency
    - Network latency is more limiting than network capacity
    - Fetch many URLs at the same time
        - To increase URLs fetched per second
    - Need threads for concurrency
- Fetch each URL only *once*
    - avoid wasting network bandwidth
    - be nice to remote servers
    - Need to remember which URLs visited 
- Know when finished



go可以检查race condition
`go run -race crawler.go`

##### locks vs channels?
- Most problems can be solved in either style
- What makes the most sense depends on how the programmer thinks
    - state -- sharing and locks
    - communication -- channels
- For the 6.824 labs, I recommend sharing+locks for state,
    and sync.Cond or channels or time.Sleep() for waiting/notification.

## Remote Procedure Call (RPC)
- a key piece of distributed system machinery; all the labs use RPC
- goal: easy-to-program client/server communication
- hide details of network protocols
- convert data (strings, arrays, maps, &c) to "wire format"
```
RPC message diagram:
  Client             Server
    request--->
       <---response

Software structure
  client app        handler fns
   stub fns         dispatcher
   RPC lib           RPC lib
     net  ------------ net

```

#####  kv.go
- A toy key/value storage server -- Put(key,value), Get(key)->value Uses Go's RPC library
- Common: (声明)
    - Declare Args and Reply struct for each server handler.
- Client: (声明)
    - `connect()`'s `Dial()` creates a TCP connection to the server
    - `get()` and `put()` are client "stubs"
    - `Call()` asks the RPC library to perform the call

- Server: (声明)
    - Go requires server to declare an object with methods as RPC handlers
    - Server then registers that object with the RPC library
    - Server accepts TCP connections, gives them to RPC library
        - The RPC library
        - reads each request
        - creates a new goroutine for this request
        - unmarshalls request
        - looks up the named object (in table create by Register())
        - calls the object's named method (dispatch)
        - marshalls reply
        - writes reply on TCP connection
        - The server's Get() and Put() handlers
        - Must lock, since RPC library creates a new goroutine for each request
        - read args; modify reply
    
#####  A few details:
- Binding: how does client know what server computer to talk to?
    - For Go's RPC, server name/port is an argument to Dial
    - Big systems have some kind of name or configuration server
- Marshalling: format data into packets
    - Go's RPC library can pass strings, arrays, objects, maps, &c
    - Go passes pointers by copying the pointed-to data
    - Cannot pass channels or functions

##### RPC problem: what to do about failures?
  e.g. lost packet, broken network, slow server, crashed server

##### What does a failure look like to the client RPC library?
- Client never sees a response from the server
- Client does *not* know if the server saw the request!
    - Maybe server never saw the request
    - Maybe server executed, crashed just before sending reply
    - Maybe server executed, but network died just before delivering reply

##### Simplest failure-handling scheme: "best effort"
  - `Call()` waits for response for a while
  - If none arrives, re-send the request
  - Do this a few times
  - Then give up and return an error

##### Q: is "best effort" easy for applications to cope with?

A particularly bad situation:
  - client executes
    - Put("k", 10);
    - Put("k", 20);
  - both succeed
  - what will Get("k") yield?
  [diagram, timeout, re-send, original arrives late]

##### Q: is best effort ever OK?
   - read-only operations
   - operations that do nothing if repeated
    - e.g. DB checks if record has already been inserted

##### Better RPC behavior: "at most once"
- idea: server RPC code detects duplicate requests
    returns previous reply instead of re-running handler
##### Q: how to detect a duplicate request?
- client includes unique ID (XID) with each request
    - uses same XID for re-send
  
- server: 
    ```
    if seen[xid]:
      r = old[xid]
    else
      r = handler()
      old[xid] = r
      seen[xid] = true
  ```
#####  some at-most-once complexities
- this will come up in lab 3
- what if two clients use the same XID?
    - big random number?
    - combine unique client ID (ip address?) with sequence #?
- server must eventually discard info about old RPCs
    - when is discard safe?
    - idea:
      - each client has a unique ID (perhaps a big random number)
      - per-client RPC sequence numbers
      - client includes "seen all replies <= X" with every RPC much like TCP sequence #s and acks
    - or only allow client one outstanding RPC at a time
      arrival of seq+1 allows server to discard all <= seq
  - how to handle dup req while original is still executing?
    - server doesn't know reply yet
    - idea: "pending" flag per executing RPC; wait or ignore

#####  What if an at-most-once server crashes and re-starts?
  - if at-most-once duplicate info in memory, server will forget
    and accept duplicate requests after re-start
  - maybe it should write the duplicate info to disk
  - maybe replica server should also replicate duplicate info

#####  Go RPC is a simple form of "at-most-once"
  - open TCP connection
  - write request to TCP connection
  - Go RPC never re-sends a request
    - So server won't see duplicate requests
  - Go RPC code returns an error if it doesn't get a reply
    - perhaps after a timeout (from TCP)
    - perhaps server didn't see request
    - perhaps server processed request but server/net failed before reply came back/n

#####  What about "exactly once"?
  unbounded retries plus duplicate detection plus fault-tolerant service