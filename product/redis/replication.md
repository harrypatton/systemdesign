#Redis Replication

##Why?
1. Scalability
2. Reliability by data redundancy.

##Features
1. Async replication.
2. A master can have multiple slaves.
3. Salve can connect to other slaves.
4. Replication is non-blocking on master. It can continue to accept queries when salves are doing replications.
5. Replication is also non-blocking on slave. In replication,
	* It can accept queries using old dataset.
	* Configurable to return error in that case.
	* After the initial sync, the old dataset must be deleted and the new one must be loaded. The slave will block incoming connections during this brief window (that can be as long as many seconds for very large datasets).
6. It is possible to use replication to avoid the cost of having the master write the full dataset to disk: a typical technique involves configuring your master redis.conf to avoid persisting to disk at all, then connect a slave configured to save from time to time, or with AOF enabled. This could be very dangerous (in the case of master dataset becomes empty.)

##Persistence turned off
It is recommend to turn on the feature. If off, be careful to set auto-restart which can make master database empty.

##Consistency
Starting with Redis 2.8, it is possible to configure a Redis master to accept write queries only if at least N slaves are currently connected to the master. However, because Redis uses asynchronous replication it is not possible to ensure the slave actually received a given write, so there is always a window for data loss.

This is how the feature works:

* Redis slaves ping the master every second, acknowledging the amount of replication stream processed.
* Redis masters will remember the last time it received a ping from every slave.

The user can configure a minimum number of slaves that have a lag not greater than a maximum number of seconds.

If there are at least N slaves, with a lag less than M seconds, then the write will be accepted.

You may think of it as a relaxed version of the "C" in the CAP theorem, where consistency is not ensured for a given write, but at least the time window for data loss is restricted to a given number of seconds.

If the conditions are not met, the master will instead reply with an error and the write will not be accepted.

##References
1. https://redis.io/topics/replication
