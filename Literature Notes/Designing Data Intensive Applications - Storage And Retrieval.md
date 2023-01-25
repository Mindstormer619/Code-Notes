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

#todo

----

## References
1. [Designing Data Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) Chapter 3