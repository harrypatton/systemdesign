#References
1. http://www.allthingsdistributed.com/2008/12/eventually_consistent.html
2. http://www.allthingsdistributed.com/2010/02/strong_consistency_simpledb.html


#Details
##Introduction
* Eventually Consistent - trade off between consistency and availability
	* score high marks in the areas of security, scalability, availability, performance, and cost effectiveness, and they need to meet these requirements while serving millions of customers around the globe, continuously.
	* when a system processes trillions and trillions of requests, events that normally have a low probability of occurrence are now guaranteed to happen and need to be accounted for up front in the design and architecture of the system. 
	* Given the worldwide scope of these systems, we use replication techniques ubiquitously to guarantee consistent performance and high availability. 

##CAP
* A system that is not tolerant to network partitions can achieve data consistency and availability, and often does so by using transaction protocols. To make this work, client and storage systems must be part of the same environment; they fail as a whole under certain scenarios, and as such, clients cannot observe partitions. 
* in larger distributed-scale systems, network partitions are a given; therefore, consistency and availability cannot be achieved at the same time. This means that there are two choices on what to drop: relaxing consistency will allow the system to remain highly available under the partition-able conditions, whereas making consistency a priority means that under certain conditions the system will not be available.
* Both options require the client developer to be aware of what the system is offering. 
	* If the system emphasizes consistency, the developer has to deal with the fact that the system may not be available to take, for example, a write. If this write fails because of system unavailability, then the developer will have to deal with what to do with the data to be written. 
	* If the system emphasizes availability, it may always accept the write, but under certain conditions a read will not reflect the result of a recently completed write. The developer then has to decide whether the client requires access to the absolute latest update all the time. There is a range of applications that can handle slightly stale data, and they are served well under this model.

##Client-Side Consistency
* Strong consistency - Assume A, B, C are different processes. After the update completes, any subsequent access (by A, B, or C) will return the updated value.
* Weak consistency - The system does not guarantee that subsequent accesses will return the updated value. A number of conditions need to be met before the value will be returned. The period between the update and the moment when it is guaranteed that any observer will always see the updated value is dubbed the ``inconsistency window``.
* Eventual consistency - a specific form of weak consistency; the storage system guarantees that if no new updates are made to the object, eventually all accesses will return the last updated value.

The eventual consistency model has a number of variations that are important to consider:

* Causal consistency. If process A has communicated to process B that it has updated a data item, a subsequent access by process B will return the updated value, and a write is guaranteed to supersede the earlier write. Access by process C that has no causal relationship to process A is subject to the normal eventual consistency rules.
* Read-your-writes consistency. This is an important model where process A, after it has updated a data item, always accesses the updated value and will never see an older value. This is a special case of the causal consistency model.
* Session consistency. This is a practical version of the previous model, where a process accesses the storage system in the context of a session. As long as the session exists, the system guarantees read-your-writes consistency. If the session terminates because of a certain failure scenario, a new session needs to be created and the guarantees do not overlap the sessions.
* Monotonic read consistency. If a process has seen a particular value for the object, any subsequent accesses will never return any previous values.
* Monotonic write consistency. In this case the system guarantees to serialize the writes by the same process. Systems that do not guarantee this level of consistency are notoriously hard to program.

Many modern RDBMSs (relational database management systems) that provide primary-backup reliability implement their replication techniques in both synchronous and asynchronous modes. 

* In synchronous mode the replica update is part of the transaction. All replica have the update before claims write is complete.
* In asynchronous mode the updates arrive at the backup in a delayed manner, often through log shipping. In the latter mode if the primary fails before the logs are shipped, reading from the promoted backup will produce old, inconsistent values. Also to support better scalable read performance, RDBMSs have started to provide the ability to read from the backup, which is a classical case of providing eventual consistency guarantees in which the inconsistency windows depend on the periodicity of the log shipping.

##Server-side Consistency
* N = the number of nodes that store replicas of the data
* W = the number of replicas that need to acknowledge the receipt of the update before the update completes
* R = the number of replicas that are contacted when a data object is accessed through a read operation

Here's the condition,

* If W+R > N, then the write set and the read set always overlap and one can guarantee strong consistency. In the primary-backup RDBMS scenario, which implements synchronous replication, N=2, W=2, and R=1. No matter from which replica the client reads, it will always get a consistent answer. In asynchronous replication with reading from the backup enabled, N=2, W=1, and R=1. In this case R+W=N, and consistency cannot be guaranteed. The problems with these configurations, which are basic quorum protocols, is that when the system cannot write to W nodes because of failures, the write operation has to fail, marking the unavailability of the system. With N=3 and W=3 and only two nodes available, the system will have to fail the write.
* How to configure N, W, and R depends on what the common case is and which performance path needs to be optimized. In R=1 and W=N we optimize for the read case, and in W=1 and R=N we optimize for a very fast write. Of course in the latter case, durability is not guaranteed in the presence of failures, and if W < (N+1)/2, there is the possibility of conflicting writes when the write sets do not overlap.
* Weak/eventual consistency arises when W+R <= N, meaning that there is a possibility that the read and write set will not overlap. The period until all replicas have been updated is the inconsistency window discussed before. If W+R <= N, then the system is vulnerable to reading from nodes that have not yet received the updates.

##Partition
Partitions happen when some nodes in the system cannot reach other nodes, but both sets are reachable by groups of clients. 

* If you use a classical majority quorum approach, then the partition that has W nodes of the replica set can continue to take updates while the other partition becomes unavailable. The same is true for the read set. Given that these two sets overlap, by definition the minority set becomes unavailable. Partitions don't happen frequently, but they do occur between data centers, as well as inside data centers.
* In some applications the unavailability of any of the partitions is unacceptable, and it is important that the clients that can reach that partition make progress. In that case both sides assign a new set of storage nodes to receive the data, and a merge operation is executed when the partition heals. For example, within Amazon the shopping cart uses such a write-always system; in the case of partition, a customer can continue to put items in the cart even if the original cart lives on the other partitions. The cart application assists the storage system with merging the carts once the partition has healed.

#Summary
* Data inconsistency in large-scale reliable distributed systems has to be tolerated for two reasons
	* improving read and write performance under highly concurrent conditions
	* handling partition cases where a majority model would render part of the system unavailable even though the nodes are up and running.
* Whether or not inconsistencies are acceptable depends on the client application. In all cases the developer needs to be aware that consistency guarantees are provided by the storage systems and need to be taken into account when developing applications. 
* Many times the application is capable of handling the eventual consistency guarantees of the storage system without any problem.
