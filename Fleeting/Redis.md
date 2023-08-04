# Redis
> Created: 2023-06-14 19:17

## Capabilities

### In memory data structures

Support for strings, hashes, lists, sets, sorted sets, streams, and more.

+ Strings. Sequence of bytes
+ Lists. Lists of strings sorted by insertion order.
+ Sets. Unordered collections of unique strings. O(1) time.
+ Hashes. Record types modeled as collections of field-value pairs.
	+ Similar to HashMaps.
+ Sorted sets. Ordered by each string's associated score.
+ Streams. Data structure that acts as an append-only log.
+ Geospatial index. Useful for finding locations within a given geographic radius or bounding box.
+ Bitmaps. Bitwise operations on strings
+ Bitfields. Efficiently encode multiple counters in a string value. Atomic operations.
+ HyperLogLog. Probabilistic estimates of the cardinality (i.e., number of elements) of large sets.
+ Extensions.

### Programmability

Lua server-side scripting, stored procedures with Redis Functions.

### Persistence

Keeps the dataset in memory for fast access, but can also persist all writes to permanent storage to survive reboots and system failures.

### Clustering

Horizontal scalability with **hash-based sharding**, scaling to millions of nodes with automatic re-partitioning when growing the cluster.

### High Availability

Replication with automatic failover for both standalone and clustered deployments.

## Use Cases

+ Real time data store
	+ Realtime applications
	+ low latency usecases
+ Caching and session storage
	+ caching DB queries
	+ complex computations
	+ API calls
	+ session state
+ Streaming and messaging
	+ stream data type
	+ high data rate ingestion
	+ event sourcing
	+ notifications

----

## References
1. https://redis.io/