This note is based on YouTube video [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://www.youtube.com/watch?v=-W9F__D3oY4)

### System Scale
1. Vertical scale. There's a ceiling.
2. Horizontal scale => which machine to visit? 

A few ideas for horizontal scale,

1. Add a load balancer.
2. It could be a DNS-like machine. A domain name associated with multiple IPs. Round robin to select the machine.
3. A machine could be busy then, so round robin is not idea.
    * partition the machine based on usage, e.g., image server, php server, file server.
    * use software or hardware load balancers.
4. User's data might be on one machine. When visit again, it goes to another machine so the data may lose.
    * save user and session data in a centralized location. called sticky sessions (can save on MySql or other storage).
    * consistent hashing could help because it still distribute every user but a user is always assigned to the same machine.
    
### Raid
* RAID 0 - two disks. Each one get partial data to achieve nearly two times performance.
* RAID 1 - two disks. Each one is a `mirror` of the other. Performance overhead but fault-tolerate.
* RAID 10 - 4 disks. Combine RAID 0 and RAID 1.

### Stick Sessions
	1. Use MySql to store sessions or cookies.
	2. Use FileShare system.
	3. How to mitigate if the shared resource is down?
		a. Replication, i.e., get two - figure out how to sync.
	4. Cookies
		a. Store everything in cookie.
		b. Store the server visited before in cookie so it always goes to the same server.
			i. Downside: cookie expiration; ip could change; (we can use another id and use mapping in backend to avoid private ip exposure; we can also move user data from one server to another server and update the mapping).

PHP/Python - compiling optimization instead of interpreting every time.

Caching
	• Html
	• MySql query cache
	• Memcached

Craiglist
	• Always return html page. It generates a html page once and stores it in cache (i.e., file server).
		○ Downside: 
			§ space; 
			§ redundant data (because of the same header, footer)
			§ A big gotcha: hard to make a change which you have to touch every file to update.

MySql - query cache

Memcached - memory cache. Store data in RAM.
	• When become big, remove some data based on LRU or other metrics.

Replication - redundant and robust
	• Master - slave mode. 
		○ Promote slave if master is down.
		○ Redundant and robust.
		○ Distribute queries across machines.
		○ If read is way more than write, slaves process read only and master handle write only.
	• Master - master mode
		○ More robust for write scenario in which one master down is ok.
		○ Downside: conflict.
	
Load balancer
	• Active - active
	• Active - passive

Partition
	• Different servers
		○ Harvard.facebook.com
		○ Mit.facebook.com
		○ …

Across Data Centers
Use DNS for load balancing.
