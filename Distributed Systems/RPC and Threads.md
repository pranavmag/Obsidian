2026-11-11 02:33

### Threads

Threads are the main tool for managing concurrency. Distributed programs have many things happening at once like a client talking to many servers, or a server handling many clients.

A thread is a unit of execution within a process and a process is a program that operates in its own memory space. Inside a process can have a single thread or multiple threads. Each thread has its own program counter, registers and stack, so each can be at a different point in the code. All the stacks live in the same address space since they are part of the same process so that means threads share memory. In Go, threads are called goroutines and `main` itself is a goroutine. 

#### Three reasons to use threads

I/O concurrency - Launch one thread per RPC and each sends its request and waits for its reply so multiple requests go on at once while other work proceeds.

Multi-core parallelism - CPU heavy goroutines run in parallel, up to the number of cores.

Periodic/background work - A goroutine that loops, sleeps and acts is much simpler than weaving checks into the main logic.

There could be some overhead though. Ten threads that sleep and do a little work cost almost nothing but a million threads doing work every millisecond can overload a machine.

#### Alternatives and context

Threads vs event-driven (asynchronous) programming

An event-driven program has a single thread running a loop that waits for any event (a timer, a key press, a packet), looks up the state for that activity in a table, and advances it one step.

Hybrid: run one event loop per core (e.g. 8 threads on 8 cores) to get both parallelism and I/O concurrency.

#### Races and locks

One classic race condition is the incrementing of a variable like `n = n + 1`. The steps would be that n is loaded into a register, then adds 1, and stores it back and if two threads do it at once they may both increment and store back the same value. If n was 0 and two threads are incrementing at the same time then the value of n is 1 but it's supposed to be 2. The result is entirely dependent on whether or not one thread finishes its store before the other one loads.

The fix is to use a mutex (lock). Whoever gets the lock first runs the whole sequence before any other threads can acquire the lock. 

The other strategy is to restructure the code so that threads don't share data at all if possible. It's simpler but not always possible.

#### Coordination

Locking is about threads staying out of each other's way but coordination is when threads interact and wait for each other. For example with a channel in Go when data is sent from one goroutine to another the receiver has to wait until data arrives from the sender.

#### Deadlock

A deadlock occurs when thread 1 waits for thread 2 while thread 2 waits for thread 1, so neither can ever proceed. For example thread 1 acquires lock A and thread 2 acquires lock B. Thread 1 then needs lock B and thread 2 needs lock A and now both threads are waiting forever because neither can release the lock that they hold and they need the lock that the other thread has.

If the program doesn't crash but is just hanging or stops making progress then look for a deadlock.





