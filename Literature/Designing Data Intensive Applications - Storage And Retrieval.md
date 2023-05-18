# Designing Data Intensive Applications - Storage And Retrieval
> Created: 2022-11-06 16:32
> #book 

## DB Data Structures

Fastest writes are through a simple append-only log. Simply add new lines to a log file. The problem is that reading from this file becomes an O(n) operation, which is terrible for large numbers of records. Regardless, many real databases do use an append-only log as a record of certain types of operations and data.

> 💡 **Index.** An additional data structure, derived from the primary data. The purpose of indices is to boost performance -- you can add or remove indexes without changing the structure of the actual data model. When you add an index, it generally makes writes a bit slower but speeds up reads along certain query directions. _Well chosen indexes speed up reads, but EVERY index slows down writes._

### Hash Indexes

Simple approach for key-value pair data. Write your data in a log-style, with data being appended to the end of the log as `(key, value)`. Then you can use an in-memory hashmap to map `key` to the offset in the log file where you can find the value. Steps for writing and retrival are:

1. For writing
   1. Append `(key, value)` to the disk log (ideally in a binary format for quick writing, retrieval, and minimum use of disk space).
   2. Update the RAM mapping for `key` to the byte offset where the value is present.
2. For reading
   1. Consult the in-memory hashmap with `key` to get the byte offset
   2. Disk-seek in the file to the offset to get the value.

_Bitcask_ (Riak's storage engine) does this, where it offers high performance reads and writes, subject to all keys fitting within the available RAM. Note that the size of the value is not similarly limited. This is suited to situations where we don't have a ton of keys, but the value may be updated many times (for example, view counts on videos. The number of videos is always going to be orders of magnitudes smaller than the number of times the view count needs to be changed).

**Avoiding running out of disk space.** We write into a "segment" file, closing it once it reaches a certain size and opening a new one to continue writing live values. Meanwhile, the old segment files are _compacted_. Compaction means that we take the files, get rid of old values, and _merge_ multiple files into one.

Consider two sequential segment files like:

```
File 1:
	cat:345 dog:453 cat:567 dog:1203 dog:1777 cat:966
File 2:
	cat:1045 dino:147 cat:1970 cat:2075
```

Each file can be compacted to (keeping only their latest values for each key and discarding old ones):

```
File 1:
	cat:966 dog:1777
File 2:
	cat:2075 dino:147
```

and can be further merged as:

```
Final compacted file:
	cat:2075 dog:1777 dino:147
```

This compaction and merging can be done in a background thread, as we _create a new file_ for each compacted file or the final merged file, during which we can continue serving read requests as normal (for example, if `dog` is queried while the final compacted file is being written, we need to read from File 1). After the merging is complete, we redirect new read requests to the final merged file, and delete the old files.

#### Real Implementation

There are some details to take care of:
1. **File format:** Prefer binary (encode length + raw string without delimiters) over formats like CSV.
2. **Deleting records:** Append a special record called a _tombstone_ that tells the compact/merge process that the key needs to be deleted when the tombstone is seen (discard old values).
3. **Crash recovery:** Periodically write the hashmap to disk (snapshot it) so that you can quickly recover from crashes rather than have to read entire large segment files.
4. **Partially-written records:** Checksums for detecting, then ignoring or correcting incorrect records in case of a partial write before a crash.
5. **Concurrency control:** One single append-only writer thread, multiple concurrent reader threads (as segment files are otherwise immutable).

> ❓ **Why append-only instead of updating the file in place?** One, appending and segment merges are *sequential write operations*, which are a lot faster on disks than random writes (and also faster to some extent on SSDs). Two, much easier to have concurrency and crash recovery. Three, merging old segments allows to avoid the problem of data fragmentation over time.

**Limitations:**

+ Hash table must fit in memory. Too many keys cannot be maintained, and it's hard to have a partially on-disk hash table.
+ Range queries are not efficient — hashmaps are good at retrieving specific keys, not entire ranges of keys.

### SSTables and LSM-Trees

SSTables are _Sorted String Tables_, where each segment file differs in that it is _sorted by key_. Advantages:
1. Easier to compact and merge. Since the keys are sorted, you can use an algorithm similar to mergesort to go through all the keys in the segments being merged, and quickly compact/merge them.
2. We do not need _all_ keys in memory. If you have the keys `handbag` and `handsome` in the memory map, you can tell that they key `handiwork` will be between them (or not at all). This way, you can sparsely store keys rather than have to store every single one.
3. Since read requests require scanning over multiple keys as-is, we can compress each "block" down and store it in the disk, reducing both space and I/O usage.

Maintaining an SSTable:
1. When a write comes in, add it to a _memtable_ — an in-memory balanced tree structure like an RBT.
2. When the memtable gets bigger than a threshold (few MB), write it out to an SSTable on disk. This new file becomes the latest segment of the DB, any incoming requests during this write can be serviced by a new memtable instance.
3. In order to serve read requests, first query memtable, then the newest on-disk segment, then the second-newest segment and so on.
4. Occasionally run compaction/merge on the older segment files, and discard overwritten or deleted values.

The only issue is that if the DB crashes, the memtable is lost. To prevent this, use an append-only log to keep track of writes to the memtable. The log isn't in sorted order, but doesn't matter because it's only used to restore the memtable in the event of a crash.

#### LSM-Trees from SSTable

The overall indexing structure, including both the memtable and the SSTable segments, is called a _Log-Structured Merge Tree_, or an LSM-Tree.

In practice, we do a few extra things:
+ _Bloom filters_ can be used to quickly tell if a key is not present in the stored data, to prevent going through every single SSTable segment looking for a missing key.
+ _Size-tiered_ and _level-tiered_ compaction strategies for determining the order and timing of merge-compacting the SSTables.

### B-Trees

![[Pasted image 20230116145120.png]]

+ B-trees divide data into fixed size pages instead of variable-length segments
+ Pages reference each other through memory references (see diagram).
+ The *branching factor* of the B-tree is the number of references at each level. This is a constant.
  + The diagram has a branching factor of 6.
  + Generally set to a few hundred for an actual implementation.
+ Inserting a key into a B-tree involves:
  + Finding the page where the data has to be inserted.
  + If the page is nearly full, then the page is split down the middle and the data inserted in one of the resulting child pages.
  + The updated midpoint reference is added as a key to the parent page.
+ A B-tree with a branching factor of 500, and 4KB pages, can store and reference data up to **256 terabytes!**

In practice, B-trees need some extra things to be implemented for reliability.

+ We use a *write ahead log (WAL)*, an append-only file which B-tree modifications are written to before the tree itself is modified. This way, if the system crashes while writing, the WAL can be used to restore the tree to a consistent state.
+ Concurrency control with latches is needed to prevent multiple threads from seeing the tree in an inconsistent state. This is not a problem with LSM-Trees, because all the merging happens in the background, on segments that are older than the one that's currently being written to actively.

#### Optimizations

+ Some databases keep a copy-on-write mechanism instead of maintaining a WAL. A modified page is written to a different location, and the parent page is pointed at the new location. Also helps in concurrency.
+ Don't store the entire key, abbreviate it. At intermediate levels, you know that the keys are between specific ranges, hence you can use that to store a subset of the key information, therefore taking less space allowing to pack more keys and have a higher branching factor with the same page size.
  + This optimization is generally called a B+ tree.
+ Adjacent leaf nodes can be laid out in a sequential order in the disk, however this is difficult to maintain over a lot of modifications to the tree.
+ Leaf nodes pointing to siblings (both left and right) can be done to avoid having to jump back to the parent in order to get the previous or next page when scanning across a bunch of sequential keys.

### LSM Trees vs B Trees

Advantages of LSM Trees:

+ B-trees write each piece of data at least twice (once to the WAL, once to the tree, maybe more depending on page split). An LSM tree also rewrites data multiple times due to compaction and merging (_write amplification_).
  + LSM trees typically have less write amplification than B-trees (depends on implementation).
+ They write more compact and sequential SSTables, rather than B-trees which write pages to (usually) random sectors of the disk.
+ They have more compressible files. B-trees show more fragmentation, especially when compared to level-compacted LSM-trees.

> 💡 **Write amplification.** One write to the database resulting in multiple writes to the disk. This can cause performance issues over the lifetime of the database if the bottleneck to performance is the disk write speed.

Downsides of LSM-Trees:

+ Compaction process for LSM-trees requires resources which can be limited on a disk, and can hence interfere with the logging process. B-trees can be more predictable.
+ On high write throughputs, it is possible that the storage engine does not throttle the writes at all (this is typically true in practice) and therefore the resources are starved for the compaction process. This will require external monitoring -- because otherwise the disk might fill up, and also fewer compacted segments means that future reads will become slower and slower.
+ Strong transactional semantics are easier in B-trees, because each key resides in exactly one place in the memory (while in LSM trees, keys are duplicated across segments and even within the written segment).

### Other Indexes

A secondary index might either reference data kept in a different location (a _heap file_) or might directly store the data linked within the index itself (a _clustered index_). An example of the latter is InnoDB, which stores the primary key as a clustered index, and then secondary keys/indices point to the primary key. This might be done because the hop from a heap file to an index is too much of a performance hit to bear.

A good midway between them is a _covering index_, which is an index which stores only _some columns_, not all. Those columns are said to be _covered_ by the index -- for the rest of the column data you need to go to the entry in the heap file.

Clustered and covering indices can speed up reads of data, but require additional storage and add overhead on writes. Also transactional guarantees on these require extra work.

#### Multi-column indices

_Concatenated indices_ are fairly common, where we simply combine the indices into a single index in some order defined by the index. For example, `(lastName, firstName)` is useful to search by the last name, or even by the full name, but it is useless for searching specifically by the first name.

Apart from this we usually use specialized data structures for indices on specific usecases (like R-trees for spatial indices).

#### Full-text and fuzzy search

Lucene is a DB engine which allows fuzzy searching. The index for the SSTable-style term dictionary used by Lucene is a finite state automaton over the characters in the keys (similar to a trie) which allows searching for words within a certain edit distance efficiently.

Full-text search also adds features for synonyms, grammatical corrections and so on. Full scope is outside the book.

### In-Memory Databases

The limitations of disks can be avoided by storing the entire database in memory, which is increasingly possible as RAM becomes cheap. The main issue with in-memory DBs then becomes durability, which can be solved by:
+ Special hardware (e.g. battery powered RAM)
+ Writing a log of changes to disk
+ Periodic snapshots to disk
+ Replicating across multiple machines

When the DB is restarted, it needs to reload state from disk or across the network from a replica. But the reads are still served entirely from memory.

The performance advantages aren't because of the read speed from the memory instead of disk -- rather it is from the fact that you don't have to create the overhead of complex data structures (like B-trees or LSM-trees) specifically for the purpose of writing to a disk.

They can also directly store data structures which are difficult to store in disk-based structures, like priority queues and sets (Redis offers this).

We can also store more data than available memory, by evicting least recently used data to the disk, and bringing it back to the memory when requested (similar to how the OS handles memory pages). However, the entire index for the data must fit in memory.

## Transaction Processing or Analytics?

_Transaction processing_, also called _OLTP_ (Online Transaction Processing), is a pattern of database access which involves low-latency reads and writes, for relatively small amounts of data. The application is generally interactive and usually centres around the latest state of data, with small random-access reads and writes.

In contrast, _online analytical processing_ (OLAP) involves accessing specific data and pulling out measures like aggregates, from a massive store of data. This involves bulk import while writing and scanning over large records while reading, and relates to historical data as opposed to latest information. This is done usually by internal business analysts to gather insights from the database.

In the past, SQL systems were generally used for both OLAP and OLTP. However, these days OLAP is being moved over to specialized _data warehouses_.

### Data Warehousing

#todo

----

## References
1. [Designing Data Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) Chapter 3
