# Designing Data Intensive Applications - Partitioning
> Created: 2023-02-23 14:52
> #book 

For very large datasets, or very high query throughput, we need to break the data into **partitions**, also known as **sharding**.

Normally, each piece of data belongs to exactly one partition. The main reason to want partitioning is scalability. Different partitions can be placed on different nodes, and so a large dataset can be distributed across many disks, with the query load distributed across many processors.

> **Partitioning and Replication.** Partitioning is usually combined with replication so that copies of each partition are stored on multiple nodes. This means that, even though each record belongs to exactly one partition, it may still be stored on several different nodes for fault tolerance. Note that a single node may store more than one partition.

## Partitioning of Key-Value Data

The goal is to spread the data evenly across the nodes. If each node takes a fair share, then theoretically we should have a linear proportion of higher write/read throughput and data storage ability.

If the partitioning is unfair, so that one or more nodes have an uneven amount of data, we call it a **skewed partitioning.** A partition with disproportionately high load is called a *hot spot*.

> The simplest approach for avoiding hot spots would be to assign records to nodes randomly. That would distribute the data quite evenly across the nodes, but it has a big disadvantage: when you’re trying to read a particular item, you have no way of knowing which node it is on, so you have to query all nodes in parallel.

### Key-Range Partitioning

Assign a continuous range of keys to each partition, like the way an encyclopedia is organized.

![[Pasted image 20230223151539.png]]

The ranges of keys are not necessarily evenly spaced, this can lead to skew. In order to distribute the data evenly, the partition boundaries need to adapt to the data. These boundaries may be chosen manually or automatically.

> Key-range partitioning is used by Bigtable, HBase, RethinkDB and MongoDB (pre v2.4)

Within each partition, we can keep keys in sorted order -- range scans become easier.

However, the downside is that certain access patterns can lead to hotspots. For example, if we store the key as a timestamp for sensor data, then all writes from the current day will go into the same partition, overloading it while others sit idle. To mitigate this, we can preface the key with the sensor ID (assuming you have a lot of sensors), which will make range keys still possible within a specific sensor, but will require querying across partitions for range access across all sensors for a specific date.

### Hash Partitioning

Because of the risk of skew and hotspots, many distributed datastores use the hash of the key to determine the partition. A good hash function takes skewed data and makes it uniformly distributed.

> 💡 **The hash function need not be cryptographically strong!** Cassandra and MongoDB use MD5, Voldemort uses the Folwer-Noll-Vo function.

Each partition can be assigned a range of hashes, instead of a range of keys. 

![[Pasted image 20230223152713.png]]

This technique is fairly good at distributing keys evenly. The partition boundaries can be evenly spaced, or they can be chosen pseudorandomly (this is called *consistent hashing*).

> 💡 **Consistent Hashing.** A way of evenly distributing load across an internet-wide system of caches such as a content delivery network (CDN). It uses randomly chosen partition boundaries to avoid the need for central control or distributed consensus. Note that consistent here has nothing to do with replica consistency or ACID consistency, but rather describes a particular approach to rebalancing.
> 
> This is rarely used in practice in databases. Ideally avoid this term and just say "hash partitioning".

Unfortunately we lose the ability to efficiently run range queries. Keys that were once adjacent are now scattered across all partitions.

> In MongoDB, if you have enabled hash-based sharding mode, any range query has to be sent to all partitions. Range queries on the primary key are not supported by Riak, Couchbase, or Voldemort.
> 
> A table in Cassandra can be declared with a compound primary key, consisting of multiple columns. Only the first part of that is hashed to determine the partition, but the other columns are used as a concatenated index for sorting the data in Cassandra’s SSTables, so you can efficiently range-scan across these columns.
> 
> The concatenated index approach enables an elegant data model for one-to-many relationships. For example, on a social media site, one user may post many updates. If the primary key for updates is chosen to be (user_id, update_timestamp), then you can efficiently retrieve all updates made by a particular user within some time interval, sorted by timestamp. Different users may be stored on different partitions, but within each user, the updates are stored ordered by timestamp on a single partition.

### Skewed Workloads and Relieving Hotspots

In the extreme case where all reads and writes are for the same key, you still end up with all requests being routed to the same partition. (This can happen, for instance, if a celebrity's tweet is getting accessed a lot due to going viral.)

Today, most data systems are not able to automatically compensate for such a highly skewed workload, so it’s the responsibility of the application to reduce the skew. For example, if one key is known to be very hot, a simple technique is to add a random number to the beginning or end of the key. Just a two-digit decimal random number would split the writes to the key evenly across 100 different keys, allowing those keys to be distributed to different partitions.

However, having split the writes across different keys, any reads now have to do additional work, as they have to read the data from all 100 keys and combine it. This technique also requires additional bookkeeping: it only makes sense to append the random number for the small number of hot keys; for the vast majority of keys with low write throughput this would be unnecessary overhead. Thus, you also need some way of keeping track of which keys are being split.

## Partitioning and Secondary Indexes

Secondary indexes are required in order to allow searching records for occurrences of a particular value -- it need not identify a record uniquely. However, they do not map neatly into partitions.

> Many key-value stores like HBase and Voldemort do not support secondary indexes. Riak on the other hand has support for them. It is also highly important in search servers like ElasticSearch or Solr.

### Document-Partitioning

![[Pasted image 20230223220951.png]]

In this case, the user request is routed to both partitions, where `color:red` is checked in the secondary index, resulting in two documents in Partition 0 and one document in Partition 1.

Each partition is completely separate -- each partition maintains its own secondary indices, mapping to the documents in that partition alone. It doesn't care what data is stored in other partitions. These types of indices are also called *local indexes* (as opposed to global indexes).

The downside is that the query needs to be sent to *all* partitions, not just one, and the results need to be combined by the client query program. This approach is often called **scatter/gather**, and it can make read queries on secondary indices rather expensive. Even if the partition queries are done in parallel, it is prone to *tail latency amplification*.

> MongoDB, Cassandra, VoltDB, ElasticSearch, SolrCloud, Riak -- all use document-partitioned secondary indices.

### Term-Partitioning

Rather than keeping a secondary index in each partition (local index), we can keep a *global index* that covers all partitions. However, this index itself needs to be partitioned -- storing it on one node will make it a bottleneck that defeats the purpose. However, this index partitioning can be different from the partitioning of the original data.

![[Pasted image 20230223221823.png]]

Above, red cars from all partitions appear under `color:red` in the index, but the index is partitioned so that colors starting with the letters *a* to *r* appear in partition 0 and colors starting with *s* to *z* appear in partition 1. The index on the make of car is partitioned similarly (with the partition boundary being between *f* and *h*).

This is called **term-partitioned** -- the term we are looking for determines the partition of the index. For example, looking for `color:red` here goes to partition 0.

The advantage of global indexes is that reads are more efficient: rather than scatter-gather across all partitions, the client queries only the partition where the term being looked for exists (and then might query specific further partitions for the data itself, but will not require to query every single partition). But now writes are more complicated -- a write to a single document may affect multiple partitions to update the indexes (see ID `893` in the image above).

In practice, we do not have distributed transactions to update the global indexes on a write synchronously -- rather we have async updates.

> Amazon DynamoDB states that its global secondary indices are updated within a fraction of a second in normal circumstances, but may experience longer propagation delays in the case of infrastructure faults.

## Rebalancing Partitions

#todo 

----

## References
1. DDIA - Chapter 6