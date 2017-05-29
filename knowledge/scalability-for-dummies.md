[Scalability for Dummies](http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones)

1. Clone the services (horizontal scale or replicate)
    * never store data in a single machine. Use sticky session. Save user data in central storage or load balancer.
2. Handle DB. 
    * Master-slave mode; partition the data; denormalization.
    * go to NoSql. Use denormalization from the begining; no join on DB level; App will do the join.
3. Cache - cache DB queries, cache object itself.
4. Asynchronism - batch processing. 
    * Precook some data, e.g., dynamic web page => static one.
    * Async task - use a queue so worker can process it later. Customer will query job status.
