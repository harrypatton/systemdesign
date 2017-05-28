Source: [How We've Scaled Dropbox](https://www.youtube.com/watch?v=PE4gwstWhmc)

## Characters
1. High write-read ratio. Almost 1:1.
2. ACID: Atomic, Consistency, Isolated and Durability.
    * Atomic: upload the file as a whole.
    * Consistency
    * Isolation: people can use it offline.
    * Durability: no data loss.
    
## System
1. start with simple. They know how to create a better system but we never know what users need.
2. MySql works very well to scale.
3. Adding multiple instances seems easy on the surface but more difficult when start doing it.
4. Customize memcached to ensure consistency. 
