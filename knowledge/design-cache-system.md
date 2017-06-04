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
