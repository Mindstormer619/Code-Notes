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



----

## References
1. [Designing Data Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) Chapter 3