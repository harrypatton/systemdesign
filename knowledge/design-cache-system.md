# Problem
Source: https://www.interviewbit.com/problems/design-cache/

## SNAKE principle
* S: Scenario (feature expectation)
* N: Needs (user, traffic, memory)
* A: Applications (services)
* K: Kilobytes (data storage)
* E: Evolove (perf, scalability, robustness)

The following analysis is my adjustment based on source information. It may not perfectly fit in SNAKE.

## Scenarios
1. How much data we need to cache? The scale of Twitter or Google.
2. What's the eviction strategy? We need to make room to accommodate new entries when cache is full.
3. cache access pattern
    * Write through cache - write succeeds when both cache and DB are updated. Write latency is higher. Good for applications when write and reread quickly (for consistency purpose).
    * Write around cache - write direclty to DB. Read from DB in case of cache miss. Faster write (because skip cache writting) but higher latency when write and then re-read quickly.
    * Write back cache - write succeeds when cache is updated. The cache then asynchronously sync this write to DB. Low write latency, high write throughput and fast read. However, we may risk lossing the data if cache crashed before writting back to DB. To improve the odd, we can write to two caches.
    
## Needs
1. QPS and traffic (or data size) are very important to calculate.
2. Say total 30TB to cache.
3. What's QPS? We need user acount in DAU and operation count to calculate. We need to think of average and peek concurrent user count and operation count.
      * This estimation is important to calculate machines we need.
      * The source shows 10m QPS. `1b people * 50 queries per day / 24 / 3600 ~, use Peek constant, so it is around 6m QPS.` Ignore why 10m QPS here. In reality, some website shows average 60k google searches per second.
4. How many machines do we need? 
      * cache has to be in memory. One main machine can have 72GB RAM.
      * `32TB / 70GB = 420 machines.` (it is minimum machine count based on memory size. We may add more machines if we cannot handle the QPS).
      
### Design golas
1. Latency - is this a latency sensitive system? Yes, otherwise why cache? another example. searching typeahead suggestions are useless if latency is high.
2. Consistency - does it require strong consistency? or eventual consistency is ok?
3. Availability - does it require 100% reliability?

Go back to cache system.

1. Latency - very important.
2. Consistency - not really. Eventually consistency is fine.
3. Availability - yes. You have to choose C vs. A. (be careful, this really depends on scenario.)

## Deep Dive
1. think about a single machine scenario? How to design a LRU cache with a single thread?
      * if we have enough memory, hash table is enough to hold the cache. Very fast.
      * if we have to evict some keys, use double linked list and hash table together.
2. What is a single machine with multiple threads?
      * it doesn't seem to help because we have to lock the linked list. In addition, RAM is already very fast.
      * **Update**
         * the source has a very interesting analysis here. It breaks down into small steps. Apparently write and read compete each other. Because cache system is read-heavey system, we need to prioritize read operation (which should avoid lock).
         * hash table implementation - if each element uses another linked list to save all collision values. The lock could be on row (element) level instead of the whole hash table level. This would be very cool. I like the analysis here.
         * quote from the source: `The key to understanding and optimizing concurrency problems lies in breaking the problem down into as granular parts as possible. As is the case with most concurrent systems, writes compete with reads and other writes, which requires some form of locking when a write is in progress. We can choose to have writes as granular as possible to help with performance. Instead of having a lock on a hashmap level if we can have it for every single row, a read for row i and a write for row j would not affect each other if i != j. Note that we would try to keep N as high as possible here to increase granularity.`
3. Now that we sort the problem on single machine, we need to shard the data (too much data to hold on a single machine).
      * 30TB needs 420 machines. Each machine needs to handle 23k QPS.
      * 4 core machine. (4 * 1000 * 1000 / 23k) microseconds = 174us. seems reasonable for RAM.
      * Use consistent hashing to distribute the request based on key.
4. What happened if one shard die?
      * increase latency because of cache miss. 
      * each shard should have multiple machines.
      * replicaiton -> inconsistency. master-slave or master-master.
