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
      * normalized data usually requires join operation which could be very expensive. Joining across machine is even worse.
      * denormalize data means all related data is on the same machine. low latency though. Downside is that we may have inconsistency. We need to make sure all data are eventually consistent.
3. How many machines? 30 machines. 
      * 3-copy is a good number for replication.
      * master - slave strategy. master for write and slave for read. If master fails, it takes time for slave to pick up as new master. It means there's a small down time period. If data is not sync'd to slave, there could be data loss. NOT acceptable.
      * master - master. we support multiple writes.
      * peer to peer. no master or slave. every node communicates with the change. Note: I think both master-mater and p2p are the same for me, unless master-master still have slave machines. in p2p model, they use quorum replication to control consistency.
