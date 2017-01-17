#Source
This post is notes for book https://www.safaribooksonline.com/library/view/designing-data-intensive-applications.

#Overview
Reliable, scalable and maintainable applications

The Internet has done so well that most people think of it as a natural resource like Pacific Ocean. - Alan Kay

Many applications today are data-intensive, as opposed to compute-intensive. The bigger problem of this application is the amount of data, the complexity and the changing speed.

The data-intensive app is built from common building blocks

* Storage the data - databases.
* Remember the result of an expensive operation - caches.
* Allow users to search data by keyword - search indexes.
* Send a message to another process, to be handled asynchronously - stream processing
* Periodically crunch a large amount of accumulated data - batch processing.

#Thinking About Data Systems
* Many data tools have emerged in recent years. The boundary of traditional categories become blurred.
* A single tool cannot support the wide-ranging requirements so the app breaks itself into multiple tasks. Each task can use a single tool. The app stitch them together.

There are many factors that may influence the design of a data system, including the skills and experience of the people involved, legacy system dependencies, the timescale for delivery, your organization’s tolerance of different kinds of risk, regulatory constraints, etc. Those factors depend very much on the situation.

* Reliability - The system should continue to work **correctly** (performing the correct function at the desired performance) even in the face of adversity (hardware or software faults, and even human error).

* Scalability - As the system grows (in data volume, traffic volume or complexity), there should be reasonable ways of dealing with that growth.

* Maintainability - Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.

#Reliability
##Intuitive ideas
1. perform functions that users expected
2. tolerate the user making mistakes or using the software in unexpected ways.
3. good performance
4. prevent unauthorized access or abuse.

##Faults
* Reliability - app continues to work correctly when things go wrong. Things go wrong are called **faults**. 
* Systems that anticipate faults and can cope with them are called ``fault-tolerant`` or ``resilient``.
	* In reality, it only makes sense to tolerate ``certain types of faults``. Tolerant for every possible faults it not feasible.
* Fault is not the same as failure. A fault is usually defined as one component of the system deviating from its spec, whereas a failure is when the system as a whole stops providing the required service to the user. It is impossible to reduce the probability of a fault to zero; therefore it is usually best to design fault tolerance mechanisms that prevent faults from causing failures. (In case of fault components - a node is down, we can still provide correct data to user to avoid failure.)
* Counter-intuitively, in such fault-tolerant systems, it can make sense to increase the rate of faults by triggering them deliberately — for example, by randomly killing individual processes without warning. Many critical bugs are actually due to poor error handling; by deliberately inducing faults, you ensure that the fault-tolerance machinery is continually exercised and tested, which can increase your confidence that faults will be handled correctly when they occur naturally. The Netflix chaos monkey is an example of this approach.
* Although we generally prefer tolerating faults over preventing faults (previously we said prevent fault from causing failures, a subtle difference. In addition, preventing faults is very hard.), there are cases where prevention is better than cure (e.g. because no cure exists). This is the case with security matters, for example: if an attacker has compromised a system and gained access to sensitive data, that event cannot be undone. 

###Hardware Faults
* Add redundancy to the individual hardware components can reduce the failure rates.
* There is a move towards systems that can tolerate the loss of entire machines, by using software fault-tolerance techniques in preference to hardware redundancy. 
	* Such systems also have operational advantages: a single-server system requires planned downtime if you need to reboot the machine (to apply operating system security patches, for example), whereas a system that can tolerate machine failure can be patched one node at a time, without downtime of the entire system.

###Software Errors
* different types. just a few examples,
	* a bug crashed all instances, e.g, Linux kernel bug regarding leap second.
	* a runaway/wild process uses up all resources.
	* dependency service becomes unresponsive.
	* cascading failures
* may hide for a long time until triggered by unusual circumstance.
* reason: software make the wrong assumptions which are true in most cases but become false in rare cases.
* solution: lots of small things to help. 

###Human Errors
Humans design, build and run the system. Humans are known to be unreliable.

What can we do?

* Design a system in a way that minimizes opportunities for error.
* Decouple the places where people make the most mistakes from the places where they can cause failures. E.g. sandbox - use all features including real data but no impact on real users.
* Testing
* Quick and easy recovery
* Monitoring
* Good management practices and training.

###Importance

###Other
Availability can be measured as: Uptime / Total time (Uptime + Downtime).

Reliability is a measure of the probability that an item will perform its intended function for a specified interval under stated conditions. There are two commonly used measures of reliability:

* Mean Time Between Failure (MTBF), which is defined as: total time in service / number of failures 
* Failure Rate (λ), which is defined as: number of failures / total time in service.

the distinction between reliability and availability: reliability measures the ability of a system to function correctly, including avoiding data corruption, whereas availability measures how often the system is available for use, even though it may not be functioning correctly. For example, a server may run forever and so have ideal availability, but may be unreliable, with frequent data corruption.[6]

##Scalability
a system's ability to cope with increased load. it needs to answer the questions: if the system grows in a particular way, what are our options to cope with the growth? how can we add more resources to handle the increasing load?

###Describing load
First, we need to succinctly describe the current load on the system; only then can we discuss growth questions (what happens if our load doubles?). Load can be described with a few numbers which we call load parameters. The best choice of parameters depends on the architecture of your system: perhaps it’s requests per second to a web server, ratio of reads to writes in a database, the number of simultaneously active users in a chat room, the hit rate on a cache, or something else.Perhaps the average case is what matters for you, or perhaps your bottleneck is dominated by a small number of extreme cases.

Twitter case is very interesting to read. Generally it has two options and both have pros and cons. Twitter creates a hybrid solution (combining both) - use solution #1 for follower #1, use solution #2 for follower #2 and then merge the results shown in home timeline.

###Describing performance
Investigate what happened when the load increases.

* keep resource unchanged, how is the performance affected?
* to keep the performance, how much resources to add?

Response time is not a single number, but a distribution of values.



In batch-processing systems, we care about throughput - the number of records we can process. In online system, response time is more important - the time between sending a request and receiving the response.

Latency vs. Response time

* Response: what clients see.
* Latency: the time waiting to be handled.

Repeat the same request, the response time may vary because latency is varied because of a lot of factors.

Need to think of response time as **not** a single number but a distribution of values we need to measure.

**average** response time is not very helpful. Percentile values are more important. Median means half request below the threshold and the other half above the number. 50%, 95%, 99% and 99.9% are common. (Amazon uses 99.9% for internal services).

High percentiles are important because they directly affect users’ experience of the service. Even it is a very small amount, the volume of requests are huge and the proportion becomes higher. Even user loads a page, it may send multiple request at the same time. On the other hand, optimizing the 99.99th percentile is going to be too expensive and not yield enough benefits.

**Percentiles** are often used in service level objectives (SLOs) and service level agreements(SLAs), contracts that define the expected performance and availability of a service. For example, a SLA may state that the service is considered to be up if it has a median response time of less than 200 ms and a 99th percentile under 1 s (if the response time is longer, it might as well be down), and the service may be required to be up at least 99.9% of the time. This sets expectations for clients of the service, and allows customers to demand a refund if the SLA is not met.

Queueing delays are often a large part of the response time at high percentiles. As a server can only process a small number of things in parallel (limited for example by its number of CPU cores),it only takes a small number of slow requests to hold up the processing of subsequent requests — an effect sometimes known as **head-of-line blocking**. Even if those subsequent requests are fast to process on the server, the client will see a slow overall response time due to the time waiting for the prior request to complete. Due to this effect, it is important to measure response times on the client side.

When generating load artificially in order to test the scalability of a system, the load-generating client needs to keep sending requests **independently** of the response time. 

###Percentile in practice
High percentiles become especially important in backend services that are called multiple times as part of serving a single end-user request. Even if you make the calls in parallel, the end-user request still needs to wait for the slowest of the parallel calls to complete. It takes just one slow call to make the entire end-user request slow. Even if only a small percentage of backend calls are slow, the chance of getting a slow call increases if an end-user request requires multiple backend calls, and so a higher proportion of end-user requests end up being slow.

###Approach for coping with load
1. Scale vertically
2. Scale horizontally

Some systems are elastic - automatically increase resources when load increases.

start-up or prototype or unproven product: adding features quickly is more important than scaling.


Even though they are specific to a particular application, scalable architectures are nevertheless usually built from general-purpose building blocks, arranged in familiar patterns.

#Maintainability
1. Operability - easy for operation teams to keep the system running smoothly.
2. Simplicity - easy for new engineers to understand and remove unnecessary complexity.
3. Evolvability - easy to make changes.

#Summary
* Reliability means making systems work correctly, even when faults occur. Faults can be in hardware (typically random and uncorrelated), software (bugs are typically systematic and hard to deal with), and humans (who inevitably make mistakes from time to time). Fault tolerance techniques can hide certain types of fault from the end user.

* Scalability means having strategies for keeping performance good, even when load increases. In order to discuss scalability, we first need ways of describing load and performance quantitatively. In a scalable system, you can add processing capacity in order to remain reliable under high load.

* Maintainability has many facets, but in essence it’s about making life better for the engineering and operations teams who need to work with the system. Good abstractions can help reduce complexity and make the system easier to modify and adapt for new use cases. Good operability means having good visibility into the system’s health, and having effective ways of managing it.






