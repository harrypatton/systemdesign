#Introduction

Asynchronous I/O is a form of input/output processing that permits other processing to continue before the transmission is complete.

* I/O is slow
* Synchronous I/O: start the access and wait for it to complete. It would block the process of a program while the communication is in progress, leaving system resources idle.
* Alternatively, it is possible to start the communication and then perform processing that does not require that the I/O be completed. Any task that depends on I/O having completed is still blocked, but other processing that doesn't depend on I/O operation can continue.
* Many OS functions exist to implement async-I/O at many levels.
* Asynchronous I/O is used to improve throughput, latency, and/or responsiveness.

#Forms
1. Process: today's process concept. 
2. Polling.
3. Select(/poll) loops.
4. Signals (interrupts)
5. Callback functions
6. More ...

#References
* wiki page: https://en.wikipedia.org/wiki/Asynchronous_I/O
