#Introduction

[Update 12/27/2016] I found a few very helpful topics to discuss the topic. See section References below.

This post is to understand node.js event loop.

Event loop is the key to handle high throughput scenario in Node.js.

Event-driven programming: application flow control is determined by events or state change. E.g., any UI-based application. The basic implementation is to have a main loop that listens for events and then calls callback functions.

EventEmitter is actually synchronous.

Node.js is a single-threaded application, but it can support concurrency via the concept of event and callbacks. Every API of Node.js is asynchronous and being single-threaded, they use async function calls to maintain concurrency. Node uses observer pattern. Node thread keeps an event loop and whenever a task gets completed, it fires the corresponding event which signals the event-listener function to execute.

In node.js there is only a single thread which executes **the code you're writing** as an application author (i.e. the Javascript parts). This does not imply there's at any time only one thread running!

Node.js itself uses libeio to perform actions (like file or network access) which can block. Libeio does use threads internally to provide an asynchronous/non-blocking API to the library consumers, so it can be integrated in an external event-loop based application (in the node.js case, libev is used as event loop).

#References
* http://code.danyork.com/2011/01/25/node-js-doctors-offices-and-fast-food-restaurants-understanding-event-driven-programming/
* http://stackoverflow.com/questions/3629784/how-is-node-js-inherently-faster-when-it-still-relies-on-threads-internally
* http://blog.mixu.net/2011/02/01/understanding-the-node-js-event-loop/
* https://www.quora.com/How-does-IO-concurrency-work-in-node-js-despite-the-whole-app-running-in-a-single-thread
* http://stackoverflow.com/questions/5599024/what-so-different-about-node-jss-event-driven-cant-we-do-that-in-asp-nets-ht
* https://www.tutorialspoint.com/nodejs/nodejs_event_loop.htm
