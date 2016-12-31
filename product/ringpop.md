
#Source
* https://ringpop.readthedocs.io/en/latest/getting_started.html#what-is-ringpop

#What is Ringpop?
The source says _it is a library that maintains a consistent hash ring and can be used to arbitrarily shard the data in your app in a way that is adaptable to capacity changes and resilient to failures_.

A few notes,

* way: maintain a consistent hash ring.
* goal: arbitrarily shard the data.
* features:
	* adaptable to capacity change.
	* resilient to failures.

The source mentions 3 core features from Ringpop

* a membership protocol - its membership protocol provides a distributed application, whose instances were once completely unaware of one another, with the ability to discover one another, self-organize and cooperate. The instances communicate over a TCP backchannel and pass information between them in an infection-style manner. Enough information is shared to allow these instances to come to an agreement, or converge, on whom the participating instances, or members, are of the distributed application.
* a consistent hash ring - with a consistent membership view, Ringpop arranges the members along a consistent hash ring, divides up the integer keyspace into partitions and assigns ownership of the partitions to the individual instances of your application. It then projects a keyspace of your choosing, say the ID range of the objects in your application, onto that same ring and resolves an owner for each ID. In the face of failure, the underlying membership protocol is resilient and automatically reassigns ownership, also known as rebalancing, to the surviving instances.
* request forwarding - requests that your application serves, e.g., create new objects, update or read or delete existing ones, may be sent to any instance. Each Ringpop instance is equipped to route the request to the correct owner with the shard key, resolve to an instance that is not the one that received the original request.

Ringpop is first and foremost an application developer's library. it is not an external system/service/infrastructure shared by others.

* Membership Protocol - uses SWIM to manage nodes.
* Consistent Hashing - uses FarmHash as its hashing function. Consistent hashing applies a hash function to not only the identity of your data, but also the nodes within your cluster that are operating on that data. Ringpop uses a red-black tree to implement its underlying data structure for its ring which provides log n lookups, inserts, and removals. Ringpop adds a uniform number of replica points per node. 
* Forwarding - If a key arriving at instance A hashes to the node, it can process it, otherwise, it forwards it. This information is forwarded using a protocol called TChannel. TChannel is a networking framing protocol developed by Uber, used for general RPC. 

##E2E Scenario
[This part](https://ringpop.readthedocs.io/en/latest/architecture_design.html#how-ringpop-works) has a very nice E2E explanation.

##Flap Damping
As an example, let’s say A pings B, and B responds. Then, in the next round of the protocol, A pings B again but this time B is down. Then in the next round, A pings B, but this time B is up again. If there’s a bad actor (a slow node that’s overwhelmed by traffic), it’s going to act erratically. So we want to evict it from the cluster as quickly as possible. The pattern of deviations between alive and suspect/faulty are known as flaps.

Flap damping is a technique used to identify and evict bad nodes from a cluster. We detect flaps by storing membership update history and penalize nodes when flap is detected. When the penalty exceeds a specified suppress limit, the node is damped. When things go wrong and nodes are removed from the hash ring, you may see a lot of shaky lookups.

##Partitions
In the original implementation of ringpop, if a cluster is split to multiple partitions, nodes in each partition declare each other as faulty, and afterward will no longer communicate. Ringpop implemented support for merging the partitions, which we call **healing**.

##Partition Healing
