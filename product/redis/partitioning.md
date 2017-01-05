#References
1. https://redis.io/topics/partitioning

#Redis Partitioning
##Why Partitioning?
Horizontal scaling for much larger data base and use many computers together.

##Partition Types 
1. Range partitioning - use key value to form a range-node mapping table. It is inefficient because we maintain a table for every kind of key value.
2. Hash partitioning - hash the key value and find the node. Consistent hashing is a preferred way and implemented by a few Redis client and proxies.

##Partition Implementation Components
It could be implement in different parts of the software stack

1. Client side partitioning - client does the partitioning work and sends request to the right node.
2. Proxy assisted partitioning - client sends request to proxy which forwards to the right node. E.g., Redis and Memcached proxy Twemproxy.
3. Query routing - client can send request to any random node. Node can forward the query to right node. Redis Cluster uses a hybrid form of query routing with the help of the client (the request is not directly forwarded from a Redis instance to another, but the client gets redirected to the right node)

##Limitation
1. The partitioning granularity is key so you cannot partition a huge key.
2. It adds complexity.

##Data Store vs Cache
When Redis is used as a data store, a given key must always map to the same Redis instance. When Redis is used as a cache, if a given node is unavailable it is not a big problem if a different node is used, altering the key-instance map as we wish to improve the availability of the system (that is, the ability of the system to reply to our queries).

1. If used as a cache, scaling up and down using consistent hashing is easy.
2. If used as a store, a fixed keys-to-nodes map is used, so the number of nodes must be fixed and cannot vary; otherwise it needs to rebalance keys between nodes.

``NOTE``: I'm not sure if I understand the data store part. Why use fixed keys-to-nodes map?

##Implementation Examples
1. Redis Cluster - preferred way
2. Twemproxy - auto partitioning among multiple Redis instances.
3. Client supporting consistent hashing.
