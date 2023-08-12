# Cassandra DB
> Created: 2023-02-22 19:38

**Cassandra** is a database that works best for writing and storing a large amount of data in a distributed system. It has extremely high performance for write-heavy workloads.

> What differs between Cassandra and relational databases is how _easily_ Cassandra runs in a distributed system and how easy it is to manage it. In fact, people joke that it’s _more difficult to run it as a single node system than as a distributed one_. Being distributed is built into Cassandra and running it in a single node makes it seem like you are wasting its potential, kinda like you are using a Lamborghini in city traffic during rush hour.

Cassandra runs as a [[Leaderless Replication|masterless]] system. Most traditional DBs rely on leader and follower (master and slave) nodes. Cassandra uses homogenous nodes without the leader/follower distinction, which allows scaling easily without having to make decisions on adding new leader nodes, or leader election systems.

#todo 


----

## References
1. [[When to use and not use Cassandra]]