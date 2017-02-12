`Replication` means keeping data copy on multiple machines. 

1. data close to end users and reduce latency.
2. increase availability - system continues to work when parts of the system failed.
3. multiple machines can serve queries so increase throughput.

The challenge of replication is how to handle data changes. 3 popular algorithms are `single leader`, `multiple leaders`, `leaderless`.

#Leaders and Followers
Each node that stores a data copy is `replica`. How to ensure all replicas have all the data eventually? Common solution is `leader based replication` (aka, `master-slave`, `active-passive`.

* One node is designated as `leader`
* Write request from users only go to `leader`
* `leader` save a copy in local and then send out `replication log` to all `followers`.
* All `followers` apply the data change in the same order as processed in `leader`.
* Read quest can go to `followers`.

## Sync vs. Async replication
* Replication may take a long time 
* Sync
	* user always get up-to-date information consistent with the leader.
	* disadvantage: write cannot be process if one node slow/failed to response. (it is more like a single-point failure.)
	* in practice, one node is sync and others are async. called `semi-sync`
* leader-based replication is often configured to be completely async.
	* if leader failed, there might be data loss.
	* advantage: system can handle process as long as leader is available.

## Setting up new followers
1. take a snapshot on leader
2. apply snapshot on follower
3. follower connects to leader and asks for all changes since the snapshot.

## Handle Node Outage
Any node could be down due to unexpected issue or planned maintenance.  The goal is to keep system running even with failed nodes.

### Follower failure - catch up recovery
Node has a local log recording the last sync up point. After recover, it requests all data change since that point and then catch up.

If the node is re-imaged,  the process is the same as adding new followers.

### Leader failure - failover
The process is below,

1. one of the followers need to be promoted to leader.
2. somehow clients knows they need to talk to new leader.
3. other followers start consuming data from new leader.

Failover can happen manually or automatically. Automatic failover consists of the following steps,

1. detect that the leader failed.
2. choose a new leader.
3. update the system to use new leader.

Issues from failover,

1. data loss when new leader doesn't have latest data.
2. after new leader is selected, the previous leader may rejoin the cluster. It may have conflict values. Common solution - discard the ones from previous leader.
3. discarding writer could be very dangerous. The book gives an example of github. It uses auto-incremental value as the key to correlate. Unfortunately the new selected leader doesn't have latest data so the key got shifted over and data disclosed to the wrong users.
4. split brain - multiple nodes believe they're the leader.
5. the right time out to declare leader is dead.
	* too long - it becomes a long time to recover the system.
	* too short - unnecessary failover because the leader may have temporary load spike or network issue.

## Implementation of replication logs

### Statement based replication
The log has every statement it executes. (I think it is a good approach but the book gives a few major issues),

1. undetermined function say now() or date. It will generate different values.
2. in some scenarios, statement execution order is important so it limits the concurrent execution. (I don't really understand the scenario)
3. statement with side-effects (again, I don't understand it)

Generally not recommended.

### Write-ahead log (WAL) shipping
Every write appends to a log file.

**Disadvantage**: the log is on very low level, so coupled with storage engine. hard to transfer when switch to different versions of storage engine or even different storage engines.

### Logical log replication
use different log formats for replications and storage engines so they're decoupled. It is also called `logical log` vs. `physical log`.

In general `logical log` describes the writing on row level.

It addresses most issues from WAL shipping solution.

### Trigger based replication
All other solutions are implemented by database system. In some scenarios that we have more control like replicate only interesting data, we can use trigger on application layer. (`trigger written in database belongs to application layer.`)

#Problems with replication lags
Goal with replications

1. failure tolerate
2. scalability
3. reduce latency

leader-based replication is `read-scaling` architecture. Cannot use fully sync replication because it would prevent writing when one node failed. It has to use async replication and then cause `inconsistent data issue` (even eventually the data becomes consistent - `eventual consistency`).

`replication lag`: the duration between the write on leader and the reflection on followers.

## Reading your own writes
write goes to leader but reader may not, so a user may write something but didn't see it when view through followers.

Solution: `read after write consistency` (aka, `read-your-writes consistency`).

**guarantee**: the user always see his latest update. no guarantee that other users always see latest update.

**Implementation** - a few approaches,

1. read data from leader when user `may` or `is able to` modify the data. E.g., only the owner can update profile so the owner always get his latest profile from `leader`. other users uses `followers` to view that user's profile. This approach needs domain knowledge. (we cannot query backend to see if the data is modified or not.)
2. track the time of last update, e.g., use `follower` if last update is more than 1 minute ago; otherwise go through `leader`.
3. client can save the timestamp (or logical timestamp) and the replica must have all data at least until that timestamp; otherwise go to other replica or write for updating.

In case of multiple datacenters, it could be tricky because it may go to another datacenter to find the leader.

Scenario: the same user on multiple devices. Update timestamp on client may not work because it is probably not the latest one. (probably save online and every client needs to call service to get the value). (The book mentions another issue, but I don't understand).

##Monotonic reads
**Issue**: a user can see things `moving backwards in time` or `something disappear`. E.g., user B made a change. replica A has the change but replica B hasn't gotten it yet. When user A visits the page, it may go to replica A and then see the change. Refresh the page, it may go to replica B and then the change from B disappears.

``Monotonic reads``: a guarantee that the user always get the most recent data `they have seen`. They will not read older data after having previously read newer data.

Solution: each user reads all data from the same replica. (different users can read from different replicas).

## Consistent prefix reads
Some writes are in order, but the reader (via follower) may get out-of-order update data. 

`This guarantee` says that if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order.

It is not a problem if database can write in order but this issue usually happens in `partition` because of no writer order. One solution, the writes related to each other go to the same partition.

## Solution for replication lag
1. Evaluate the impact of replication lag. 
2. Transaction on distributed system could be the answer. (The book has a separate chapter for that).

# Multi-leader replication
Multiple nodes can accept writes. master-master. When one master node processes the data, other leaders act as `follower` to that `master`.

## User Cases
### Scenario - multi-datacenter operation.

1. single-leader solution is inefficient for users who are not in the same data center.
2. in multi-leader replication, one datacenter has one leader.
	* inside the data center, it is more like single-leader model.
	* the leader will send the change to another leader in different data center.
	* the other leader will propagate changes to all followers in the same data center.

**Advantages**
 * good write performance by reducing inter-datacenter latency
 * tolerate datacenter outage

**big downside: conflicts.**

### Scenario - client with offline operation
local database acts as a leader. Multiple devices => multiple leaders. Conflict happens. Replication lag could be days.

### Scenario - collaborative editing
change commits to local storage and then merge.

## Handling write conflicts
a single-leader database, it doesn't have conflict. In multi-leader database, the conflict is only detected `asynchronously` at some later point.

### Conflict Avoidance
Common approach: all writes for particular records go to the same leader.

Sometimes you cannot avoid conflicts.

### Converging towards a consistent state
The database must resolve the conflict in a `convergent` way, which means that all replicas must arrive at the same final value when all changes have been replicated.

A few ideas

1. Give each write an id and the one with highest id wins. When we use time stamp as id, it is known as `last write wins`. 
2. Give each replica an id. The write from replica with highest id wins.
3. Merge values together so no data loss.
4. Record all values in separate data source and ask user to resolve the conflict.

### Custom Conflict Resolution Logic
We can write code to resolve the conflicts.

1. on write - when system writes and detects the conflict, it calls the code to resolve conflict.
2. on read - system saves all data when conflict. When read, all versions of data return and the app needs to resolve.

## Multi-leader replication topologies
It is used when more than 2 leaders,

1. `all to all`: a leader sends the write to every other leader.
2. `circular topology`: a leader sends its writes to next one.
3. `start topology`: one designated root node forwards all writes to other nodes.

Be careful about `concurrent writes`, `version vector` can be used. (`time stamp` is not sufficient).

# Leaderless replication
A few databases like Dynamo use this solution. They're called `dynamo-style`.

## Writing to the database when a node is down
leader-based configuration - fail over is necessary.

In leaderless configuration - read and write can send to all nodes. The operation is considered success as long as `m` nodes accept, e.g., `w+r > n` (quorums), each value has a version and we return the one with higher version.

### Read Repair and Anti-entropy
when an unavailable node comes back online, it needs to catch up on writes it missed,

1. Read repair: a client reads value from multiple nodes in parallel. It may find stale value from one node and then write new value back to that node.
2. Anti-entropy process: background process looks for difference and fixes them.

## Quorums for reading and writing
if there are `n` replicas, every write must be confirmed by `w` nodes to be considered successful, and we must query at least `r` nodes for each read. 

As long as `w + r > n`, we expect to get an up-to-date value when reading, because at least one of the r nodes we’re reading from must be up-to-date. Reads and writes that obey these r and w values are called `quorum reads and writes`. `r` and `w` as the `minimum` number of votes required for the read or write to be valid.

In Dynamo-style database, these numbers can be configurable. A common choice is,

1. `n` is a odd number (machines)
2. both `r` and `w` set to `(n+1)/2` (round up).

`the quorum condition w+r > n` can tolerate the following failures,

1. if `w<n`, we can still process writes if one node failed.
2. if `r<n`, we can still process reads if one node failed.
3. With n = 3, w = 2, r = 2 we can tolerate one unavailable node.
4. With n = 5, w = 3, r = 3 we can tolerate two unavailable nodes. 
5. Normally, reads and writes are always `sent to all n replicas in parallel`. The parameters w and r determine how many of the n nodes need to report success before we consider the read or write to be successful.

when fewer than `r` or `w` nodes available, the operation failed.

### Monitoring Staleness
For leader-based replication, database exposes the metric for replication logs. Checking the point between leader and followers can give an idea about the replication lag.

harder to monitor leaderless based replication.

## Sloppy quorums and hinted handoff
Quorum can tolerate failed nodes without fail over. It can also tolerate slow nodes (because they just need minimum `r` or `w` nodes' response).

It is good for applications that require high availability and low latency but ok with occasionally stale reads.

In a large cluster, a network error may cause a client to lose connections to a lot of nodes that cannot reach the quorums. Trade-off,

1. return error.
2. Or should we accept writes anyway, and write them to some nodes that are reachable but aren’t among the n nodes on which the value usually lives?

the latter is called `sloppy quorums`: writes and reads still require `w` and `r` successful responses, but those may include nodes that are not among the `designated n home nodes` for a value. 

when network is fixed, the write on temporary nodes are sent back to designated ones. this is called `hinted handoff`.

It is useful to increase writing availability.

## Multiple datacenter operation
both multi-leader and leaderless replication can be used in multi-datacenter operations.

## Concurrent writing
For defining concurrency, exact time doesn’t matter: we simply call two operations concurrent if they are both unaware of each other, regardless of the physical time at which they occurred. 

Because of problems with clocks in distributed systems, it is actually quite difficult to tell whether two things literally happened at the same time.

