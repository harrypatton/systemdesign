
#Hash
* A hash function is any function that can be used to map data of arbitrary size to data of fixed size. 
* Widely used in hash table for rapid data look up.
* They are also useful in cryptography because it is hard to reconstruct the original data from hashed value.

#Uses
## Hash Table
Hash table - quickly locate a data record given its search key. https://en.wikipedia.org/wiki/Hash_table
* The domain of possible keys is larger than its range (hash table size), so multiple keys may map to the same slot. This is called conflicts.
* A slot is called bucket, and the hash value is called bucket indices.
* hash function tells where to start looking for the record. A good hash function can narrow the search down to one or two entries.
* Collision resolution: all methods require that both key and value stored in the table.
* Process: given a key, get the hashed value and then find the bucket. Compare each element key from the bucket with the key.

## Other Uses
* Caches - hash function used to build caches for large data set. **I'm not sure if I understand this part**, probably use hash function to check if a key exists or not.
* Bloom filters
* Finding duplicate records
* Protecting data
* Finding similar records
* Finding similar substrings

#Properties
A good hash function is required to satisfy certain properties below.

1. Determinism - for a given input value it must always generate the same hash value.
2. Uniformity - map expected inputs as evenly as possible over its output range.
3. Non-invertible

#Q&A
1. What's the hash value type? A 32-bit integer covers 2^32 range which is not big enough for distributed hash table.
  * There are two steps. First find the located node - a 32-bit integer should be enough for that. Once locate the node, a 32 or 64-bit integer is sufficient for all keys stored on that node.

#Refer
* https://en.wikipedia.org/wiki/Hash_function
