#Consistent hashing

A naive solution could be pick up a random server by hash-value % node-count. It works fine until you add or remove a new node. Because of node-count change, everything is hashed to a new location.

It would be nice if, when a cache machine was added, it took its fair share of objects from all the other cache machines. Equally, when a cache machine was removed, it would be nice if its objects were shared between the remaining machines. This is exactly what consistent hashing does - consistently maps objects to the same cache machine, as far as is possible, at least.

The basic idea behind the consistent hashing algorithm is to hash both objects and caches(nodes) using the same hash function. The reason to do this is to map the cache(node) to an interval, which will contain a number of object hashes. If the cache(node) is removed then its interval is taken over by a cache(node) with an adjacent interval. All the other caches(nodes) remain unchanged.

Think about it as a circle. To find the location of a key, hash the key, and find next value representing a node on the circle (by clockwise direction). To avoid hot spot on some nodes, every node has a lot of replica virtual nodes so it spreads over the circle.

##Follow-up questions
1. What’s the best way to cope with servers of different sizes?
2. How to add and remove more than one machine at a time?
3. How to cope with replication and fault-tolerance?
4. How to migrate data when jobs are going on (including backups)?
5. How best to backup a distributed dictionary?

#Distributed Hash Table (DHT)
A dictionary to store <k1, v1>, <k2, V2> across a cluster of computers. The API is easy to use so user doesn't need to think about the details of cluster.

#Reference
##Consistent Hashing
* http://www.tom-e-white.com/2007/11/consistent-hashing.html

##Distributed Hash Table
* http://www.mikeperham.com/2009/01/14/consistent-hashing-in-memcache-client/
