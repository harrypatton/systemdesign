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
* xxx
* xx
