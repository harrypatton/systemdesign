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

``One issue`` - data loss because the memtable is lost when crashed. To avoid the problem, each write will also append to a file on dish (order doesn't matter, because it is for recovery). when memtable is written back to disk, remove that log file.

### Other
Lucene - an indexing engine for full-text search used by ElasticSearch and Solr. It uses SSTable-like sorted files.

## B-trees
1. Most common type of index.
2. Used as standard index implementation in almost all relational DBs, and many non-relational DBs.
3. Like SSTables, it also keeps key-value pairs sotred by key, but here's the differences,
	* The log-structured indexes earlier break the database down into variable-size segments, typically several megabytes or more in size, and always write a segment ``sequentially``. 
	* B-trees break DB into fixed-size blocks or pages, usually 4kB in size, and read or write a page a a time. Corresponds more closely to underlying hardware.
4. One page is designated as the root of B-tree. When look up a key, start here. It contains k keys and key+1 references to child pages. Each child is responsible for a continuous range of keys and the keys in root page indicate the range. A leaf page contains key and the value (or the reference to pages where value could be found). 

When update or insert new key-value, the tree remains balanced. A tree with n keys always has a height of O(logn). 

### Update-in-Place vs. Append-only Logging
B-tree is to overwrite a page on disk with new data. It assumes overwrite doesn't change page location. In contrast, log-structured indexes only appends to file, but never modify files in place.

Think of overwriting a page on disk as an actual hardware operation. 

Some operations require several different pages to be overwritten (e.g., one page is full so we have to split). Corrupted index happens when DB crash after write only some of the pages. So it needs an addtional data structure - ``write-ahead log``. It is an append-only file to which every tree modification must be written before the actual page overwrite. It is used to restore if DB crashed.

## B-Trees vs. LSM-Trees
B-Trees
* more mature implementation
* faster reads
* a key exists only once so better for transactional semantics.

LSM-Trees
* better performance
* high throughput of random writes
* Compaction may impact performance of ongoing read and write. A higher percentiles of quite high response time. B-tree can be more predictable.

## Storing Values Within Index
The value could be the row itself or a reference to where row stored. The latter case is called ``heap file``. It is good because all indexes refer to the same value data. If new value is bigger than old one and needs to move heap file to somewhere, so indexes have 2 options,
* update all indexes to new location
* in old heap file, add a location reference to new heap file. called ``hop``

Sometimes ``hop`` is too expensive so it stores entire row in index. This is known as ``clustered index``.

A compromise between a clustered index (storing all row data within the index) and a non-clustered index (storing only references to the data within the index) is known as a ``covering index`` or ``index with included columns``, which stores some of a table’s columns within the index.

As with any kind of duplication of data, clustered and covering indexes can speed up reads, but they require additional storage and can add overhead on writes. Databases also need to go to additional effort to enforce transactional guarantees, because applications should not see inconsistencies due to the duplication.

## Multi-column Indexes
multi-column indexes and multi-dimensional indexes (e.g., geospatial data, rgb color).

## Fuzzy Indexes
search similar keys (e.g., misspelled words). Lucene is able to search text for words within a certain ``edit`` distance. (an edit distance of 1 means a letter is added, removed or replaced.)

## Keeping everything in memory
disk is too slow but two significant advantages
* durable
* cheap

RAM becomes cheaper and many datasets are not too big => in memory DB. Every read is from memory. Write is persisted on disk by append-only log.

The performance gain is not only from RAM access (because OS can cache recent used disk blocks), but also avoiding the overhead of transferring data in disk format on write.

It can also provide data models that are difficult to implement with disk-based indexes.

# Transaction Processing or Analytics?


