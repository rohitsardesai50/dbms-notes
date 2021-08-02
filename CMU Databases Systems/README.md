Course Link: https://www.youtube.com/playlist?list=PLSE8ODhjZXjbohkNBWQs_otTrBTrjyohi


# Relational Model #

## Disk oriented architecture ##
 - Slotted pages
 - Log structured orgnanization
   - HBase, Cassandra, LevelDB, RocksDB, S3
   - Build indexes, compact log.

## Tuple storage ##
  - Tuple is seq of bytes. DBMS interprets bytes into attributes and values
  - Overflow storage pages
  - External storage(Blob) (no durability/transactions)

## OLTP v/s OLAP ##
### OLTP  ###
Simple queries read/update small amount of data
### OLAP ###
Queries read large amounts of data spanning multiple entities

## n-ary Storage Model(NSM) ##
-  DBMS stores all attributes for a single tuple contiguously in a page.

* Advantages
   * Fast inserts, updates, and deletes.
   * Good for queries that need the entire tuple.

* Disadvantages
   * Not good for scanning large portions of the table and/or a subset of the attributes.

## Decomposition Storage Model(Columnar) ##
The DBMS stores the values of a single attribute for all tuples contiguously in a page.
   * Ideal for OLAP workloads where read-only queries perform large scans over a subset of the table’s attributes.

* Advantages
  * Reduces the amount wasted I/O because the DBMS only reads the data that it needs.
  * Better query processing and data compression (more on this later).
* Disadvantages
  * Slow for point queries, inserts, updates, and deletes because of tuple splitting/stitching.

* Examples
  * SybasIQ, Vertica, Druid, Clickhouse, Pinot, MonetDB

## Buffer Pool Manager ##
Buffer pool is a memory region organized as an array of fixed-size pages called frames.
Page table keeps track of which pages are currently in memory.

### OS page cache
DBMS bypass Page cache using direct IO

###  Buffer Pool Eviction Policies
- LRU
- Clock ( Approximate LRU )
- Susceptible to sequential flooding

## Access Methods
How DBMS execution engine reads/writes pages.

###  Hash Tables

Static Hashing

Allocate a giant array that has one slot for every element you need to store.
To find an entry, mod the key by the number of elements to find the offset in the array.
e.g: MD5, MurmurHash, Google FarmHash etc.

STATIC HASHING SCHEMES
Approach #1: Linear Probe Hashing

Single giant table of slots.
Resolve collisions by linearly searching for the next free slot in the table.
→ To determine whether an element is present, hash to a location in the index and scan for it.
→ Have to store the key in the index to know when to stop scanning.
→ Insertions and deletions are generalizations of lookups.

Approach #2: Robin Hood Hashing
Variant of linear probe hashing that steals slots from "rich" keys and give them to "poor" keys.
→ Each key tracks the number of positions they are from here its optimal position in the table.
→ On insert, a key takes the slot of another key if the first key is farther away from its optimal position than the
second key.

Approach #3: Cuckoo Hashing
Use multiple hash tables with different hash function seeds.
→ On insert, check every table and pick anyone that has a free slot.
→ If no table has a free slot, evict the element from one of them and then re-hash it find a new location.
Look-ups and deletions are always O(1) because only one location per hash table is checked.

Dynamic Hashing
The previous hash tables require the DBMS to know the number of elements it wants to store.
→ Otherwise it has rebuild the table if it needs to grow/shrink in size.
Dynamic hash tables resize themselves on demand.

Chained Hashing
Maintain a linked list of buckets for each slot in the hash table.
Resolve collisions by placing all elements with the same hash key into the same bucket.
→ To determine whether an element is present, hash to its bucket and scan for it.
→ Insertions and deletions are generalizations of lookups

Extendible Hashing
Chained-hashing approach where we split buckets instead of letting the linked list grow forever.
Multiple slot locations can point to the same bucket chain.
Reshuffling bucket entries on split and increase the number of bits to examine.
→ Data movement is localized to just the split chain.

→ Linear Hashing
The hash table maintains a pointer that tracks the next bucket to split.
→ When any bucket overflows, split the bucket at the pointer location.
Use multiple hashes to find the right bucket for a given key.
Can use different overflow criterion:
→ Space Utilization
→ Average Length of Overflow Chains

### Table Indexes
Hash tables are good for point lookups with sequential scans. For range scans we need table indexes.
A table index is a replica of a subset of a table's attributes that are organized and/or sorted for efficient access using a subset of those attributes.
The DBMS ensures that the contents of the table and the index are logically in sync.

It is the DBMS's job to figure out the best index(es) to use to execute each query.
There is a trade-off on the number of indexes to create per database.
→ Storage Overhead
→ Maintenance Overhead

#### B+Trees
There is a specific data structure called a B-Tree.
People also use the term to generally refer to a class of balanced tree data structures:
→ B-Tree (1971)
→ B+Tree (1973)
→ B*Tree (1977?)
→ B link-Tree (1981)


A B+Tree is a self-balancing tree data structure that keeps data sorted and allows searches, sequential access,
insertions, and deletions in O(log n).
→ Generalization of a binary search tree in that a node can have more than two children.
→ Optimized for systems that read and write large blocks of data.


The original B-Tree from 1972 stored keys + values in all nodes in the tree.
→ More space efficient since each key only appears once in the tree.
A B+Tree only stores values in leaf nodes. Inner nodes only guide the search process.

B+TREE VISUALIZATION
https://cmudb.io/btree
Source: David Gales (Univ. of San Francisco)

Clustered Indexes
The table is stored in the sort order specified by
the primary key.
→ Can be either heap- or index-organized storage.
Some DBMSs always use a clustered index.
→ If a table doesn’t contain a primary key, the DBMS will automatically make a hidden row id primary key.
Other DBMSs cannot use them at all.

INTRA-NODE SEARCH
Approach #1: Linear
→ Scan node keys from beginning to end.
Approach #2: Binary
→ Jump to middle key, pivot left/right depending on comparison.
Approach #3: Interpolation
→ Approximate location of desired key based on known distribution of keys.

