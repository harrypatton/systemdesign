#Redis Cluster

##101
Redis Cluster provides a way to run a Redis installation where data is ``automatically sharded across multiple Redis nodes``.

* The ability to automatically split your dataset among multiple nodes.
* The ability to continue operations when a subset of the nodes are experiencing failures or are unable to communicate with the rest of the cluster.

##TCP ports
two ports

* 6379: normal port used to serve clients.
* 16379: used for node-2-node communication channel using binary protocol for failure detection, configuration update etc.

##Data Sharding
* Doesn't use consistent hashing
* It uses fixed hash slot - 16384 total
* ``this part I'm confused, moving hash slots does take time and data may miss`` - Because moving hash slots from a node to another does not require to stop operations, adding and removing nodes, or changing the percentage of hash slots hold by nodes, does not require any downtime.

##Master-slave model
* Automatically create Slave node for master.
* If master failed, slave is promoted to master.
* If both slaves and master failed, the system is down.

##Consistency Guarantee
* Not able to guarantee strong consistency.
* Write may lose if master crashed before sending data to replication.
* Write may lose if partition occurred and a master is involved in a minority of instances. Current master accepts write but the slave in another partition is promoted to master. With ``node timeout``, the previous master can stop accepting queries.

##References
1. https://redis.io/topics/cluster-tutorial
