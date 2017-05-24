This note is based on YouTube video [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://www.youtube.com/watch?v=-W9F__D3oY4)

### System Scale
1. Vertical scale. There's a ceiling.
2. Horizontal scale => which machine to visit? 

A few ideas for horizontal scale,

1. Add a load balancer.
2. It could be a DNS-like machine. A domain name associated with multiple IPs. Round robin to select the machine.
3. A machine could be busy then, so round robin is not idea.
    * partition the machine based on usage, e.g., image server, php server, file server.
4. User's data might be on one machine. When visit again, it goes to another machine so the data may lose.
    * save user and session data in a centralized location. called sticky sessions (can save on MySql or other storage).
    * consistent hashing could help because it still distribute every user but a user is always assigned to the same machine.
    
### Raid
* RAID 0 - two disks. Each one get partial data to achieve nearly two times performance.
* RAID 1 - two disks. Each one is a `mirror` of the other. Performance overhead but fault-tolerate.
* RAID 10 - 4 disks. Combine RAID 0 and RAID 1.

