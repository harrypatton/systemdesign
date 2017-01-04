#References
1. https://en.wikipedia.org/wiki/MapReduce
2. https://www.quora.com/What-is-Map-Reduce

#Map Reduce
A programming model for distributed computing where tasks are distributed to various compute nodes that have access to local data via a distributed file system (ie. HDFS). Results are then consolidated if necessary and written to the distributed file system again. 

A MapReduce program is composed of a Map() procedure (method) that performs filtering and sorting (such as sorting students by first name into queues, one queue for each name) and a Reduce() method that performs a summary operation (such as counting the number of students in each queue, yielding name frequencies). 

##Map
