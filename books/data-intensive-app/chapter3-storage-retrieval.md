# Storage and Retrieval

It is about how storage engine works. It helps developer to choose the right storage engine appropriate for the app.

## Hash indexes
### Features
* use index in memory (i.e., hash table) to store mapping "key => seek location on file". 
* All keys fit in RAM.
* good for frequency updated keys (otherwise too many keys to fit in RAM)

### What happened when running out of disk space?
* break the log into segments of a certain size.
* compaction on segments - remove duplication and keep the most recent.
* compaction makes segment much smaller so merge segments, write into new segment and remove old ones.
* each segment has its own hash table in memory.
* to find a key, find most recent segment. if not found, go to second-most-recent segment and until found or scan all segments.

### Why append-only log instead of update?
* much faster than random writes.
* concurrency and crash recovery are much simpler because it is append-only or immutable.
* easy to merge segment.

### limitations
* All keys must fit in RAM. on-disk hash map has bad performance.
* No way to get keys in a range. Still have to check each key individually.

## SSTables and LSM-trees
Previously segment is a sequence of key-value pair with the order that they were written and most recent value wins.

``New format``: key-value pair is ``sorted by key``. This format is called ``sorted string table`` or ``SSTable``. Each key appears only once in each segment file. **Note**, it doesn't use append-only anymore, why?

A few advantages over ``hash indexes``
* Merging segment is simple and efficient, even files are bigger than RAM. Think about merge sort.
* No need to keep an index of all the keys in memory. It stores some keys in sparse so we can use binary search to find the key.

### How to get data sorted by key in the first place?
A few data structures like red-black trees or AVL trees that we can insert keys in ``any`` order and read back in ``sorted`` order.

Here's the workflow,

1. when write comes in, add to in-memory balanced tree (aka memtable).
2. when memtable gets bigger than some threshold, write out to disk as SSTable file. When file write is done, empty memtable.
3. when read, find in memtable => most recent on-disk segment => next-order segment
4. from time to time, a background to compact and merge segments.

One issue - data loss because the memtable is lost when crashed. To avoid the problem, each write will also append to a file on dish (order doesn't matter, because it is for recovery).

