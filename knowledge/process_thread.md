Both node.js and redis use single thread to improve throughput and performance. One of main reasons is the cost of process and thread. This topic will check how expensive to create a process and thread.

##Process

##Thread
* It is cheaper than creating a process.
* Creating thread could be costly if your work is short.
* A new thread allocates at least 1mb memory. This memory is different than GC so the collection could be expensive.
* If there're too many threads, a lot of time may spend on context-switch.
* Thread is an OS concept so it involves some OS calls.
* Java: http://stackoverflow.com/questions/5483047/why-is-creating-a-thread-said-to-be-expensive?rq=1
* .NET: http://stackoverflow.com/questions/5626803/why-is-creating-a-new-thread-expensive
* .NET thread vs. thread pool: http://stackoverflow.com/questions/230003/thread-vs-threadpool?rq=1

## Process vs. Thread
### Reference
* http://stackoverflow.com/questions/200469/what-is-the-difference-between-a-process-and-a-thread

Each process provides the resources needed to execute a program. A process has a virtual address space, executable code, open handles to system objects, a security context, a unique process identifier, environment variables, a priority class, minimum and maximum working set sizes, and at least one thread of execution. Each process is started with a single thread, often called the primary thread, but can create additional threads from any of its threads.

A thread is the entity within a process that can be scheduled for execution. All threads of a process share its virtual address space and system resources. In addition, each thread maintains exception handlers, a scheduling priority, thread local storage, a unique thread identifier, and a set of structures the system will use to save the thread context until it is scheduled. The thread context includes the thread's set of machine registers, the kernel stack, a thread environment block, and a user stack in the address space of the thread's process. Threads can also have their own security context, which can be used for impersonating clients.

The typical difference is that threads (of the same process) run in a shared memory space, while processes run in separate memory spaces.

Taken from "Parallel and Distributed Programming Using C++" by Cameron Hughes and Tracey Hughes, Table 4-1:

What is the difference between threads and processes?
The major differences between threads and processes are:

* Threads share the address space of the process that created it; processes have their own address space.
* Threads have direct access to the data segment of its process; processes have their own copy of the data segment of the parent process.
* Threads can directly communicate with other threads of its process; processes must use interprocess communication to communicate with sibling processes.
* Threads have almost no overhead; processes have considerable overhead.
* New threads are easily created; new processes require duplication of the parent process.
* Threads can exercise considerable control over threads of the same process; processes can only exercise control over child processes.
* Changes to the main thread (cancellation, priority change, etc.) may affect the behavior of the other threads of the process; changes to the parent process do not affect child processes.

### Context switch
thread context switch and process context switch: https://www.quora.com/How-does-thread-switching-differ-from-process-switching
