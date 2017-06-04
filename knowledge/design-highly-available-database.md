# Problem
Source: https://www.interviewbit.com/problems/highly-available-database/

Design a distributed key value store which is highly available and is network partition tolerant

## Scenario
Based on CAP theorem, we choose availability over consistency.

1. data size: 100TB
2. do we support key-value update? Yes
3. value size change? yes. new value may have bigger size.
4. Can a single value doesn't fit in one machine? no. up to 1GB.
5. QPS for the DB? assume 10k.

## Needs
1. how many machines? 72GB and 10TB per machine. we need minimum 10 machines. We need more machines later for replication and achieve the QPS goal.
2. Design Golas
    * Consistency: no; but we can aim eventual consistency.
    * Availablity: yes
    * Network partition tolerance or latency: yes. Sometimes people uses Latency here. Low-latency is a requirement for many systems.
    
## Deep Dive
1. Is shard required? yes. one machine cannot fit all data.
2. should the stored data be normalized? I don't know the answer. depending on the scenario. 
    * normalized data usually requires join operation which could be very expensive.
