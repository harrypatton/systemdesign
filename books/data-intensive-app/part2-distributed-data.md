#Distributed Data
Why?

1. Scalability - need to support bigger volume and read/write load than a single machine can handle.
2. Availability - replicate data to be fault tolerant
3. Latency - serve users at various location

``Vertical scaling`` cost is super-linear. limited to a single geo-location.

it is also called ``horizontal scaling``. Each machine is called a ``node``.

Replication vs. Partitioning
* Replication: data copy on different nodes. Provide redundancy, fault tolerance and improved performance.
* Partitioning: a big one split into many subsets. Partially improve availability and performance.

It is good to mix replication and partitioning.
