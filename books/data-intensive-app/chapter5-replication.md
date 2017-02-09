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
