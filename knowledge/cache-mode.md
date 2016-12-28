#Cache Mode
When a system writes data to cache, it must write the data to backing store at some point. The time of writing back is controlled by write policies,

1. [write-through](https://en.wikipedia.org/wiki/File:Write-through_with_no-write-allocation.svg): both writes are done simultaneously. 
2. [write-back (write behind)](https://upload.wikimedia.org/wikipedia/commons/c/c2/Write-back_with_write-allocation.svg): initially write to cache only. When the cache is to updated/modified by another cache, the writing to backing store occurred. This way is more complex to implement.
	* A read miss in a write-back cache (which requires a block to be replaced by another) will often require two memory accesses to service: one to write the replaced data from the cache back to the store, and then one to retrieve the needed data.

Write-miss: write a data which may not exist in cache, thus there are two approaches for situations of write-misses:

1. Write allocate (also called fetch on write): data at the missed-write location is loaded to cache, followed by a write-hit operation. In this approach, write misses are similar to read misses.
2. No-write allocate (also called write-no-allocate or write around): data at the missed-write location is not loaded to cache, and is written directly to the backing store. In this approach, only the reads are being cached.

Both write-through and write-back policies can use either of these write-miss policies, but usually they are paired in this way:

1. A write-back cache uses write allocate, hoping for subsequent writes (or even reads) to the same location, which is now cached.
2. A write-through cache uses no-write allocate. Here, subsequent writes have no advantage, since they still need to be written directly to the backing store.

##Cache Stale
1. Entities other than the cache may change the data in the backing store, in which case the copy in the cache may become out-of-date or stale. 
2. Alternatively, when the client updates the data in the cache, copies of those data in other caches will become stale. 

Communication protocols between the cache managers which keep the data consistent are known as [coherency protocols](https://en.wikipedia.org/wiki/Cache_memory#Cache_coherency).

#Reference
