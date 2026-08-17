2026-08-16 00:03

Tags: 

Threads are one way to achieve concurrency. They allow us to execute two control flows at the same time. The 'main' thread is where our program starts and on top of that we can have additional threads that perform tasks while our main thread also executes.

Each process can have multiple threads and these threads share the same code, data, and kernel context. Each of these threads have a unique thread id (TID) along with its own logical flow and its own stack for local variables.

```
#include <iostream>
#include <thread>

void test(int x) {

std::cout << "hello from thread!\n";

std::cout << "argument passed: " << x << '\n';

}

int main() {

std::thread myThread(&test, 100);
myThread.join();

std::cout << "hello from main thread\n";

return 0;
}
```

```
Output:
hello from thread!
argument passed: 100
hello from main thread
```

Depending on where the `myThread.join()` is placed, the threads execute in order. So if i placed `myThread.join()` after the main thread line then the output would be

```
Output:
hello from main thread
hello from thread!
argument passed: 100
```

We basically make the main thread wait until myThread has finished.

### Multiple std::thread

```
#include <iostream>

#include <thread>

#include <vector>

  

int main() {

  

auto lambda=[](int x) {

std::cout << "hello from thread " << std::this_thread::get_id() << '\n';

std::cout << "argument passed: " << x << '\n';

};

  

std::vector<std::thread> threads;

for (int i{}; i < 10; ++i) {

threads.push_back(std::thread(lambda, i));

threads[i].join();

}

  

std::cout << "hello from main thread\n";

  

return 0;

}
```

```
Output:
hello from thread 140321313715904
argument passed: 0
hello from thread 140321313715904
argument passed: 1
hello from thread 140321313715904
argument passed: 2
hello from thread 140321313715904
argument passed: 3
hello from thread 140321313715904
argument passed: 4
hello from thread 140321313715904
argument passed: 5
hello from thread 140321313715904
argument passed: 6
hello from thread 140321313715904
argument passed: 7
hello from thread 140321313715904
argument passed: 8
hello from thread 140321313715904
argument passed: 9
hello from main thread
```

As you can see here the argument passed did increment each time but the thread id is the same. So are we launching 10 threads here or is it the same thread? I figured that it was because we each thread after finishing its task was deleted so each new thread would just take the same thread id because the previous thread no longer existed. I seemed to be right on that. We did launch 10 threads here but they're launched one after the other waiting for each to terminate. Changing these few lines can change the output drastically.

```
std::vector<std::thread> threads;

for (int i{}; i < 10; ++i) {

threads.push_back(std::thread(lambda, i));

}

  

for (int i{}; i < 10; ++i) {

threads[i].join();

}
```

```
Output:
hello from thread 140701141497536hello from thread 
argument passed: 0
hello from thread hello from thread 140700981130944
argument passed: 5
hello from thread 140701124712128
hello from thread 140701091141312
argument passed: 7
140701133104832
hello from thread argument passed: hello from thread 1407010827486081
140701116319424hello from thread argument passed: 8


argument passed: 1407010995340163
140701107926720
argument passed: 4

argument passed: 6
argument passed: 2
hello from thread 140701074355904
argument passed: 9
hello from main thread
```

As we can see here it is a lot more cluttered and each thread isn't necessarily outputting in the order that they were created. The only thing that is ensured is that all of these threads must finish executing before the main thread executes. It's up to the operating system on when it schedules each thread.

### std::jthread

```
std::vector<std::jthread> jthreads;

for (int i{}; i < 10; ++i) {

jthreads.push_back(std::jthread(lambda, i));

}
```

jthread follows the RAII pattern so it calls join automatically right at the end of the scope in which it is terminated.

### Data Races 

```
static int shared_value{};

  

void shared_value_increment() {

++shared_value;

}

  

int main() {

  

std::vector<std::thread> threads;

for (int i{}; i < 100; ++i) {

threads.push_back(std::thread(shared_value_increment));

}

  

for (int i{}; i < 100; ++i) {

threads[i].join();

}

  

std::cout << "Shared value: " << shared_value << '\n';

  

return 0;

}
```

```
Output:
bash-5.3$ ./build/main
Shared value: 100
bash-5.3$ ./build/main
Shared value: 100
bash-5.3$ ./build/main
Shared value: 99 // Different Value!
bash-5.3$ ./build/main
Shared value: 100
bash-5.3$ ./build/main
Shared value: 100
bash-5.3$ ./build/main
Shared value: 100
bash-5.3$ ./build/main
```

As we can see, the output is non-deterministic. This indicates a data race. In this case multiple threads may have tried writing at the same time so they both only incremented by one even though two increments were called. 

We can use something called a mutex. A mutex is mutual exclusion and in the context of our threads it means that when one thread accesses the shared value, only that thread can access that value (read/write).

```
std::mutex gLock;

static int shared_value{};

void shared_value_increment() {

gLock.lock();

++shared_value;

gLock.unlock();

}
```

This now locks the mutex (blocks if unavailable) and all the other threads will wait for that lock to be available. The lock is protecting that shared value. Any code within the lock is the critical piece of code that needs synchronization, any code outside of the lock does not enforce synchronization.

### Deadlock

```
void shared_value_increment() {

gLock.lock();

try {

++shared_value;

throw "dangerous...abort";

} catch (...) {

std::cout << "handle exception";

return;

}

gLock.unlock();

}
```

In this case if an exception occurs the lock is acquired by a thread but never returned, so a deadlock would occur. The other threads cannot make progress because they are blocked or not able to acquire a resource.

We can place another `unlock()` right before the `return` but that starts to get a little verbose if we have multiple instances where we need to do that.

There is something we can use known as std::lock_guard which is a mutex wrapper that provides a convenient RAII-style mechanism for owning a mutex for the duration of a scoped block. When a `lock_guard` object is created, it attempts to take ownership of the mutex it is given. When control leaves the scope in which the `lock_guard` object was created, the `lock_guard` is destructed and the mutex is released.

```
void shared_value_increment() {

std::lock_guard<std::mutex> lockGuard(gLock);

try {

++shared_value;

throw "dangerous...abort";

} catch (...) {

std::cout << "handle exception";

return;

}

}
```

```
Output:
handle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle exceptionhandle...
```

Now here we see that even in the case of an exception, all threads are executed.

