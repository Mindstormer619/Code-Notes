# Designing Data Intensive Applications - Replication
> Created: 2023-01-24 19:52
> #book

_Replication_ is keeping multiple copies of data in different places (multiple machines) connected over a network.

Why?
+ Data can be kept geographically close to users (low latency).
+ To keep the system working even if parts have failed (high availability).
+ Scaling out machines for serving more queries (increased throughput).

## Leaders and Followers

_Replica:_ A node that stores a copy of the database. Eventually, all data is expected to be exactly the same on all replicas (_eventual consistency_).

Leader-based replication:
1. One of the replicas is designated the _leader_. Clients send write requests exclusively to the leader, which first writes the data to local storage.
2. Other replicas are called _followers_. When the leader writes data locally, it also sends an update to all followers as part of a _replication log_ (aka _change stream_). Each follower takes the log from the leader and updates its own local copy in the order mentioned in the log.
3. When a client wants to read data, it can query it either from any replica. Writes are accepted only on the leader, however.

### Synchronous / Asynchronous Replication

*Synchronous replication* is when the client gets back an OK for the data that it inserted only after it has been confirmed to be replicated by the followers. In other words, the leader sends the replication log to the followers, the followers confirm to the leader that they have written the data, and only then the leader responds back to the client saying the write was confirmed written.

*Asynchronous replication* is when the client gets back an OK once the leader has written the data to its own local storage. Followers are updated asynchronously, which means that when the client gets back the confirmation, some followers may not have received the latest piece of data.

It is highly impractical for _all_ followers to be synchronous — a single follower being unreachable would mean that the data is never written, and the whole system would grind to a halt. In practice, we often write *one* follower synchronously (in the event that the leader dies, this guarantees that there exists at least one follower which has all the data written to the leader and can therefore be elected as the new leader), but other followers are replicated async. This configuration is often called _semi-synchronous_.

### Setting up new followers

As new data keeps getting written to the system, setting up a new follower means that you might actually have to lock writes to the entire system while copying existing data to the new follower. However, there are approaches to do this without downtime:
1. Take a consistent snapshot of the leader's data at *some point in time*, without taking a lock on the whole DB if possible (several DB engines support this, as it is required for backups).
2. Copy the snapshot to the new follower node.
3. The follower connects to the leader and requests a log of all the changes that have been made *since the snapshot* — the snapshot needs to have a position associated with the replication log (Postgres calls it _log sequence number_).
4. When the follower has processed the backlog of data changes, we say it has _caught up_.

### Handling Node Outages

#### Follower Failure: Catch-up recovery

Similar to the section in [[#Setting up new followers]], we keep a log of the data changes in each follower that are being written. When the crashed follower comes back up, it connects to the leader and requests a log starting at the point of the last log it has received, proceeding to catch up to the latest changes.

#### Leader Failure: Failover

This is trickier — one of the followers needs to be promoted to the new leader, clients need to start contacting the new leader for writes, and other followers need to start consuming changelogs from the new leader. This process is called *failover*.

1. **Determine the leader has failed.** No foolproof way to do this. Usually nodes bounce messages back and forth, and when a node doesn't respond for a certain period of time, it is assumed to be dead.
2. **Choose a new leader.** This could either happen through an *election process*, or a new leader could be appointed by a previously elected *controller node*. This is a *consensus problem*.
3. **Reconfigure the system to use the new leader.** Clients need to start sending data to the new leader. If the old leader comes back, the system needs to ensure the old leader now realizes it's a follower and recognizes the new leader.

Things that can go wrong:
+ On async replication, the new leader may not have received all the writes from the old leader. If the old leader comes back, what happens to the writes? (Usually they are discarded, but that means worse durability of data).
+ Discarding writes can be extra dangerous if external systems (remote caches, for instance) need to be coordinated (because some data from your DBs flows into them).
+ *Split brain.* Two nodes can both believe they are the leader. If there is no process for resolving this, data can become lost or corrupted, because each leader receives a portion of the writes. Some systems have mechanisms to shut down one node when two leaders are detected — but if poorly designed it might shut down both nodes.
+ What's the right timeout before you declare the leader dead? Too long and recovery is slow, too quick and there will be unnecessary failovers. If the system is already struggling with high load or network glitches, unnecessary failovers will make the situation worse.

No easy solutions to this — failover is a hard problem. Some ops teams prefer to do failovers manually even when the system might be automatable, for this reason.

### Replication Log Implementation

#### Statement Based Replication

Every write request is sent to the followers — for a SQL DB, this means every `INSERT` / `UPDATE` / `DELETE` statement is sent. Every follower parses those requests from the log as though they were sent by a client.

Issues with this:
+ Nondeterministic functions like `NOW()` or `RAND()` will generate different values on each replica.
+ Things like autoincrementing columns need to be carefully updated in the same order on each replica.
+ Side-effects (like triggers) need to be deterministic.

Generally this leads to so many edge cases to take care of, that other replication methods are preferred.

#### WAL Shipping

The *write-ahead log* is an append-only sequence of bytes which contains every write to the DB. This log is shipped over to the followers, which build the exact data structures present on the leader from the log info.

The main disadvantage is that the WAL is very low-level — contains details of which bytes were changed in which disk blocks. Replication becomes closely coupled to the storage engine, making it difficult to change storage engines (or have different storage engines on some replicas). This can make zero-downtime upgrades of storage engines across nodes impossible, as all of the storage engine versions might need to match for replication to work.

#### Logical Log Replication

Using different log formats for replication and the storage engine — the replication log is decoupled from the storage engine internals. This is called a _logical log_. Contains per-row information:
+ Inserted rows: contains all values for all columns
+ Deleted row: contains enough info to uniquely identify the deleted row (e.g. primary key).
+ Updated row: enough info to uniquely identify the modified row, as well as new values for all modified columns.

These are generally also easier for external applications to parse (backups, caching, warehousing etc.)

#### Trigger Based Replication

If you require any custom behavior (e.g. only replicating some columns or a part of the data), then generally the replication logic is moved up to the application layer instead of letting the database directly handle all of it.

A trigger is a point where you can register custom application code which automatically executes when a data change of a specific kind happens in the DB. This can be used to set custom replication logic, and although it is more error-prone and generally slower than direct DB-handled replication, it is useful due to flexibility.

## Replication Lag

On _read-scaled architectures_ like the one described, followers are replicated asynchronously. When this happens, a read request to a follower might be out of date, because the system as a whole is only _eventually consistent_. In general, there is no real limit to how long the replication lag can be, and can be as high as several seconds or minutes.

### Reading Your Own Writes

When the user views the data shortly after making a write, they need to be able to see that the data exists. Consider what would happen when the user writes the data (it's sent to the leader), and then immediately follows it up with a read request, which hits a follower which has not caught up yet. Then it looks to the user as though the data they submitted was lost.

![[Pasted image 20230125191630.png]]

In this situation, we need _read after write consistence (RAW)_ which is also called _read-your-writes consistency_. Here, other users' updates might not be visible for some time, but the user should always be able to immediately view data they submitted themselves. Solutions:
+ When reading something the user may have modified, read it _from the leader_, else you can go to a follower. For example, a user's social media profile can only be edited by the user themselves, so any read requests for the user's own profile should go to the leader, while other users' profiles go to a follower.
+ In case most things are editable by the user, the above is not useful. In that case, other strategies can be used — e.g. for one minute after the last update, make all reads for this user from the leader. Or else replication lag on all followers can be monitored, and any followers older than one minute will not service read requests.
+ The client can send across a timestamp of its most recent write with each read request, and the system directs the read to a replica which is sufficiently up to date. This timestamp can either be a _logical timestamp_ (like a log sequence number) or an actual system clock (in which case clock sync becomes critical).

Cross-device RAW consistency is even more difficult, as the system must recognize that the different devices are the same user (info entered from one device must be visible when read-requested from a different device).
+ Approaches that remember the timestamp of the user's last update become difficult, because code running on one device will not know what updates ran on the other device.
+ If replicas are distributed across different datacenters, there is no guarantee that connections from different devices will be routed to the same datacenter.

### Monotonic Reads

Async replication can cause issues where it appears that reads are moving _backward in time_.

![[Pasted image 20230125194651.png]]

Consider where user A is reading data written by user B. User B makes an update, the first read request from A to the system is routed to a follower that's up-to-date, and user A sees the latest data that user B wrote. The _second_ read request from A to the system happens to be routed to an out-of-date follower, and therefore the data from B _disappears_, which can be disconcerting. In this case it might have even been better if the first read did not surface the new data by user B.

_Monotonic reads_ is a guarantee that this does not happen — if a user makes several reads in sequence, they will not see time go backward. i.e. data which was previously read will continue to be served on future requests.

One way of achieving this is to make sure every user makes their reads from the same replica — replica might be chosen by hash of the user ID. However, if that replica dies, the request will need to be routed to another replica.

### Consistent Prefix Reads

![[Pasted image 20230125195717.png]]

This problem generally occurs only in partitioned (sharded) databases, where writes might be registered out of order, due to each partition having its own leader. The _consistent prefix read_ guarantee says that if a sequence of writes happen in a certain order, then anyone reading those writes will also see them in the same order.

One solution is to make sure that any writes which are causally linked are written to the same partition. This might be difficult to do efficiently in some applications.

## Multi-Leader Replication

In leader-based replication, there is only one leader, so if you can't connect to it for some reason, you cannot write to the DB. In a _multi-leader configuration_ (master-master or active/active replication), we have more than one leader which can accept writes. Each leader simultaneously acts as a follower to the other leaders.

### Use Cases

#### Multi-datacenter operation

If you have datacenters distributed across multiple locations, it doesn't make much sense to have single-leader configurations, as all writes will need to go through the datacenter that contains the leader.

In a multi-leader configuration, you can have a leader in *each* datacenter. Each datacenter's leader replicates its changes to the leaders in the other datacenters.

Comparison:
+ **Performance.** For single-leader, every write must go over the internet to the datacenter with the leader. This can add significant latency to writes. Multi-leader can mean that writes are processed in the local datacenter, and replicated async to other datacenters. This improves the perceived performance.
+ **Tolerance of datacenter outages.** Each datacenter in a multi-leader config can continue operating independently of each other, so if a datacenter fails, it can catch up with the replication when it comes back online.
+ **Tolerance of network problems.** Single-leader configs are very susceptible to network issues between datacenters, as every write has to flow through the datacenter that contains the leader. This is less of an issue with multi-leader configs, where a temporary network interruption does not prevent writes from being processed.

The big downside of multi-leader replication is that the same data may be concurrently modified in two different datacenters, and this will cause *write conflicts* which need to be resolved. There are often subtle config pitfalls and surprising interactions with database features (e.g. autoincrementing keys, triggers, integrity constraints), as multi-leader replication is often retrofitted into regular databases.

#### Clients with offline operation

A client which needs to continue to work when disconnected from the internet — e.g. calendar app on a mobile device — requires multi-leader replication to service both read and write requests at any time. The changes need to be synced with a server and other devices, when the device is next online. The replication lag may be hours or even days. This can be a very tricky thing to make work correctly. [CouchDB](https://couchdb.apache.org/) is designed for this mode of operation.

#### Collaborative Editing

Etherpad and Google Docs allow realtime collaborative editing. To guarantee no editing conflicts, the application must obtain a lock on the document before the user can edit it. However, locking the entire document makes it equivalent to a single-leader config with transactions on the leader. So we might make the unit of change (the piece that's "locked") very small (e.g. a single keystroke) and avoid locking. This might require conflict resolution, however.

### Handling Write Conflicts

The biggest problem with multi-leader replication is that write conflicts are possible, and that requires conflict resolution. This problem does not occur in a single-leader config, as writes are executed in order on a single leader.

#### Conflict Avoidance

If you can ensure that all writes for a particular record go through the same leader, then conflicts cannot occur. Avoiding conflicts thus becomes the simplest strategy.

For example, if the user can edit their own data, you can ensure that requests from a particular user are always routed to the same datacenter. Different users may have different "home" datacenters, but from any one user's point of view, you only have single-leader replication.

However, when you need to change the designated leader for a record (datacenter loss, user moved to a different location etc.), conflict avoidance breaks down.

#### Converging towards a consistent state

The idea is to resolve conflicts by having all replicas arrive at the same final value when all changes have been replicated, i.e. resolving conflicts in a *convergent* way. This means we cannot simply apply the writes in the same order in which each replica received them (as different replicas in different datacenters might see the writes in a different order, and eventually we expect all replicas to become completely consistent).

For convergent conflict resolution:
+ Give each write a unique ID, pick the write with the highest ID as the "winner", throw the other writes away. This is called *last write wins (LWW)*. Popular approach, but dangerously prone to data loss.
+ Give each replica a unique ID, let writes that originated at a higher numbered ID take precedence. This also implies data loss.
+ Somehow merge the values together e.g. order them alphabetically and then concatenate them?
+ Record the conflict in a separate data structure that preserves all information, and then let the application/user decide what to do to resolve the conflict.

#### Custom conflict resolution logic

+ **On write.** Execute a custom conflict handler which occurs as soon as the DB system detects a conflict in the log of replicated changes. The handler typically runs in the background quickly, and cannot prompt the user. E.g. Bucardo.
+ **On read.** When a conflict is detected, all the conflicting writes are stored (see the last point in the previous section). The next time the data is read, all the conflicts are returned, and the app may prompt the user or automatically resolve the conflict and write the result back to the DB.

Conflict resolution typically happens per-row, not per-transaction.

> **Automatic Conflict Resolution.** There is interesting research for automatically resolving conflicts without custom code.
> + **CRDTs** (conflict-free replicated datatypes) are a family of data structures which can be concurrently edited by multiple users and automatically resolve conflicts in sensible ways. See https://crdt.tech/.
> + **Mergeable Persistent Data Structures** track history and use three-way merges, like Git.
> + **Operational Transformation** is the conflict resolution algo behind Google Docs, which is designed particularly for concurrent editing of an ordered list of items, like the list of characters in a text document.

### Topologies

![[Pasted image 20230126003627.png]]

+ *All-to-all* topology has every leader send writes to every other leader.
+ *Circular* topology has nodes forward writes to the next node.
+ *Star* topology has one designated "root" node forward writes to all of the other nodes.

Both circular and star topologies can have issues of network connection failures — all-to-all does not face the same issue because it allows messages to travel along different paths, avoiding single points of failure.

But even all-to-all can have issues with writes arriving at other leaders out of order, which causes conflicts of causality (see [[#Consistent Prefix Reads]]). Conflict detection and resolution in multi-leader architectures can be complex — be aware of what the system is capable of, read documentation and test the database thoroughly.

## Leaderless Replication

If we allow *any replica* to accept writes from the client directly, this is a leaderless system. An example is Amazon's internal [Dynamo][dynamo]. 

![[leaderless.svg]]

### Writing to the database when a node is down

There is no concept of failover in leaderless configs. If a node goes down, we may continue writing to the other replicas. The client sends a write request to all replicas in parallel, and we consider the write as successful if a certain quorum of replicas respond with a write confirmation. When the dead node comes back online, reads from that node may produce stale data.

To solve that problem, *reads are also sent to several nodes in parallel*. The client gets different responses from different nodes, and version numbers are used to determine which value is newer.

#### Read repair and anti-entropy

Processes done to ensure that all replicas eventually become consistent — when an unavailable node comes back online, it needs to catch up on writes that it missed.

+ **Read repair.** When a client makes several read requests in parallel, it can detect stale responses. It writes the newer value back to replicas which have stale values. This works well for values that are frequently read.
+ **Anti-entropy process.** Some datastores keep an additional background process that constantly looks for differences in data between replicas, and copies over newer data from one to the other. There may be significant delays before data is copied, however.

#### Quorums for reading/writing

If there are $n$ replicas, every write must be confirmed by $w$ nodes, and we must query at least $r$ nodes for each read, such that $w + r > n$ (the _quorum condition_). In this case, we will expect an up-to-date value for reading, because at least one of the $r$ nodes we are reading from must be up-to-date. These are called *quorum* reads or writes. The values of $r$ and $w$ here can be thought of as votes for the read/write to be valid. The values themselves can be configured according to the data load.

> **Why does the above work?** You can read the above as $w > n - r$, which means that we are writing to more nodes than the _remaining nodes_ after subtracting the ones we read from. This means that there has to be an overlap in the nodes written to and the nodes read — i.e., there has to be at least one node which is read, which was part of the latest batch of nodes written to.

Normally, reads and writes are sent to all nodes in parallel, the params $w$ and $r$ tell us how many nodes to wait for.

### Limitations of Quorum Consistency

You can keep a smaller $w$ and $r$ than quorum — which will result in faster reads and writes (and higher availability) from the cluster, but will increase the probability of stale values being read.

However, even with the quorum condition, there are other edge cases depending on the implementation:
+ If a _[[#Sloppy Quorums and Hinted Handoff|sloppy quorum]]_ is used, the writes may end up on different nodes than the reads, with no overlap.
+ If two writes happen concurrently, it is not clear which one happened first. In this case you may need to [[#Handling Write Conflicts|merge the concurrent writes]]. If winners are picked from timestamps, then we can lose writes to clock skew.
+ If a write happens concurrently with a read, the write may have reflected only on a few replicas at the time of the read.
+ If a write succeeded on some replicas but failed on others, but overall succeeded on fewer than $w$ replicas, it is _not rolled back_ on the replicas where it was written. The write itself is reported as "failed", but subsequent reads may or may not return values from that write.
+ If a node with the new value dies, and gets restored from a replica which has an old value, the number of nodes with the latest value can fall below $w$.
+ Other timing related issues are possible.

Therefore, in practice, quorums aren't a silver bullet. In particular, we do not get the guarantees discussed in [[#Replication Lag]], and those stronger guarantees require transactions or consensus.

#### Monitoring staleness

Monitoring for stale data is easier in leader-based replication, because you can simply subtract the positions of the followers' latest update with the state of the leader to find the replication lag (the writes are applied in the same order between the leader and the follower).

In leaderless systems, there is no fixed order in which writes are applied, making monitoring for lag a lot harder. Also, if the DB uses only read-repair without anti-entropy, there's theoretically no limit to how old a value may be. 

### Sloppy Quorums and Hinted Handoff

Leaderless replication is appealing for high availability low latency applications, with some level of tolerance towards stale data. However, quorums are susceptible to network interruptions which can cut off a large number of nodes from a particular client — leaving that client unable to connect to enough nodes to form the quorum, thereby completely cutting off reads/writes for that client. Even though the nodes are alive, for that particular client they might as well be dead.

In a large cluster, it is likely that the client can connect to _some_ database nodes, but the database nodes it can connect to are not necessarily part of the $n$ nodes which are the nodes on which the data lives, and therefore cannot assimilate enough nodes to form a quorum. In this case, we have a choice:
+ Is it better to return errors for all read/write requests which cannot reach a quorum?
+ Or should we accept writes anyway, and write them to some nodes that are reachable, but not part of the $n$ nodes on which the data usually lives?

The latter is called a _sloppy quorum_ — writes and reads still need the w/r quorums, but the votes can come from nodes that aren't a part of the $n$ "home" nodes for a value. (Analogy: if you lock yourself out of your house, you may go to your neighbor and ask them if you can stay on their couch temporarily.)

Once the network interruption is fixed, the temporary writes are then handed off to the "home" nodes which came back online — this is called a _hinted handoff_ (your neighbor tells you to go back home once your key is found).

Sloppy quorums increase write throughput, as you can find your $w$ write nodes even outside of the original $n$ home nodes which house the value. But this changes the quorum condition — even if the condition is satisfied, you might get stale data because the latest value may have been (temporarily) written to nodes outside of $n$. Therefore it isn't really a quorum at all — it's just an assurance of durability: your writes are stored on some $w$ nodes _somewhere_. There is no guarantee a read of $r$ nodes will see it until the hinted handoff is complete.

----

## References
1. Designing Data Intensive Applications - Chapter 5

[dynamo]: https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html