# When to use and not use Cassandra
> Created: 2023-02-22 18:27

Cassandra works really well if you want to write and store a large amount of data in a distributed system, don’t care much about ACID with good performance.

One of the most popular features of Cassandra is it can run as a distributed system.

But that may not seem a big differentiating factor between Cassandra and a traditional RDBMS since most databases these days can run in a distributed environment.

What differs between Cassandra and relational databases is how _easily_ Cassandra runs in this distributed system and how easy it is to manage it. In fact, people joke that it’s _more difficult to run it as a single node system than as a distributed one_. Being distributed is kind of built into Cassandra and running it in a single node makes it seem like you are wasting its potential, kinda like you are using a Lamborghini in city traffic during rush hour.

Another popular feature of Cassandra is how it runs as a masterless system. Most traditional databases rely on master nodes and slave nodes. Slave nodes would generally be used for replication and master nodes for writing.

This often creates complexities and difficulty in scaling the system. When you want to add more nodes, you need to make decisions on how many master nodes to add and how many slave nodes to add. Adding or removing nodes can create a lot of changes in the system as well. If a master node becomes unavailable, there would generally be a master election where a new node becomes a master.

A system which relies on homogenous nodes without this distinction of masters and slaves seems much simpler.

## Masterless replication and HA

Traditional SQL databases scale in two ways, the first is Replication or a master-slave architecture. The idea is simple enough, you add replica databases that simply hold the same data that your _master_ database does. These databases may be running as different processes, on different machines, or even in different corners of the world.

One of these is the _master_ database. This database is the only one that can perform writes, others simply replicate the data this master database holds to serve clients.

This means you have a higher reading capacity(due to each server having its own memory, CPU, and network bandwidth) with no change to your writing capacity.

This is a great thing for most applications since most applications tend to be read-heavy(think of how many tweets you read vs how many tweets you write).

The other option is to have multiple master databases or a master-master, or multi-master database.

Both these solutions generally present significant challenges. The first one, with multiple replicas and a single master, presents a single point of failure, i.e. your master database. And, since you have a single master database, it also means you cannot scale your write performance.

Adding more master nodes solves the last of those issues however it creates more challenges as well. Imagine what if two users wrote the column _salary_ on different master databases on the same row at the same time! Who would decide which data is correct!

Or what if you store one table in one database and the other in another database, how would you perform joins on tables that are in two different databases? How would you have foreign keys in one table in one server that refers to a key in another database, maybe even running on a different continent?!

Sharding, partitioning, and replication in databases is a huge topic and I may be oversimplifying some things here, so I’ll avoid diving deeper into it. But in short, scaling traditional SQL databases is difficult, even if you add multiple master nodes. This is because SQL databases have complex and strict requirements, such as ACID, validation, etc.

**So, how does Cassandra do it?**

It relies on a masterless model. In this model, all nodes actually behave the same, each storing a subset of the data. There is no _master_ or _slave._

If you were to insert a new row in the database, it will go to at least one of these nodes and get replicated to a certain number of nodes.

When you ask for your row, the nodes _gossip_ among one another to find out who holds that piece of data and return it to you. In fact, all nodes can serve requests for any piece of data, even if they don’t actually hold it. All of this is actually managed by the database nodes in the cluster with no intervention from us. The nodes in the cluster are _aware of the cluster,_ they know they are running in a distributed environment and constantly talk to each other(In fact, they talk so much to each other that this kind of talking is called [_gossiping_](https://en.wikipedia.org/wiki/Gossip_protocol)).

> Redis also shares a similar gossiping protocol. I also wrote about Redis not long ago where I wrote an entire post on Sharding in Redis [here](https://medium.com/geekculture/horizontally-scaling-writes-with-redis-clusters-a77cdcdf6de2). I also have a list of Redis posts I have written that start by explaining the basics of Redis, to replication, sharding and finally to deploying a working production level Redis cluster on AWS [here](https://medium.com/@sanilkhurana7/list/redis-88a4c95150fa). Check these out if you find posts like these interesting.

## Lack of ACID support

Databases in this sort of distributed environment would require constant data exchange among each other for strong ACID support. For example, if you want strong consistency, then all nodes should be synced with one another and should always hold the same data. If you need to add a new row or make an update to a node, then it first needs to wait for all the nodes to make that update and all of them must do it together.

Sending data from one database node to another can be time-consuming. Notice how some of the nodes aren’t even on the same continent! Apart from that, networks are complex and notorious for being unreliable.

Simply put, this sort of constant data syncing and communication isn’t reliable in such a distributed system, especially when you are getting thousands of requests per second(if not more!) and your databases are running tens of thousands of miles away from each other.

This essentially translates to making trade-offs on ACID.

**Cassandra on the other hand embraces a system with weak ACID support.** In short, it offers some level of atomicity, consistency, isolation and durability but not quite the full package that a relational database would be able to provide.

It offers partition-level atomicity but doesn’t offer rollbacks if the write operation succeeds in some nodes and fails in others. Also, depending on how you configure Cassandra, it is possible for your cluster to be in an inconsistent state. So nodes may disagree with each other on the data they hold.

In short, its support for ACID is quite weak and if ACID matters to you, you probably should reconsider using Cassandra.

## Tuneable consistency

Cassandra offers a tunable consistency model. In this model, Cassandra allows the developers to choose which level of consistency they want. You can choose between eventual consistency or strong consistency at the cost of availability.

Cassandra is usually described as an AP system, it is generally used in part due to its high availability.

However, you can control or tune the balance between availability and consistency. It is impossible to achieve both in any system as the [CAP theorem explains](https://en.wikipedia.org/wiki/CAP_theorem) so the only thing you can do is trade one with the other.

The way you control consistency is by using consistency level and replication factor. These dictate how many times your data is replicated, how many nodes must write the data synchronously before the response is returned to the user and how many nodes must return the data when reading.

Explaining how this works in-depth would be a complete post on its own so I would avoid diving deeper into it right now, in short, Cassandra is usually used as an eventually consistent distributed system, _but it can be tuned and configured to support stricter levels of consistency._

## High write performance

Since you would generally have a lot of nodes in a Cassandra cluster and each node can perform writes, you get a good write performance. In fact, [this Netflix blog post](https://netflixtechblog.com/benchmarking-cassandra-scalability-on-aws-over-a-million-writes-per-second-39f45f066c9e) explains how they tested it for 1 million writes per second with no signs of slowing down.

This level of write performance is simply not possible with a single master and multiple slave architecture in a relational database. And using a multi-master system, as we previously discussed, comes with complexities.

## Lack of a relational schema

Cassandra doesn’t support a complex relational schema. So it doesn’t support foreign keys, relationships, and other features that are common in relational databases.

Tables in Cassandra are more barebones than relational databases. Querying also comes with its own quirks. Your data design impacts what queries you can run and their performance as well. For example, you cannot (you technically can but you _really_ shouldn’t) query a table without the partition key of the table. This partition key is the key Cassandra will use to distribute your data in multiple nodes, so a query without the partition key would essentially mean that all the rows in all the nodes would need to be scanned. This, as it may seem, is not a good thing and is generally not practical for a production-level system

There is a lot more nuance to data modelling in Cassandra that I won’t dive deeper into it right now but the TLDR is you need to think of what queries you will be running on the system before your start modelling your data.

## What problems does Cassandra solve?

### Resource constraints for a single node system

If your entire database system is running on a single node, then that also means it is limited to as much CPU power, as much memory, and as much network bandwidth that can be attached to a single node.

This, needless to say, is less than optimal. Databases are heavy bulky systems, which open a lot of files, read and write a lot and are generally pretty busy and bulky guys. If I were to anthropomorphize a database, I’d probably think of Mr Incredible.

Apart from this, a single node also means a single point of failure. Eventually, at some point, your instance is going to go down. Whether it is due to a network partition, some bug in your code running on the instance, or a mess up from AWS. If your database is down and it’s not 2010 when applications could run offline, your app is also going to be down, which means your clients/company isn’t making money, which means you will get a call at 2 AM on a Saturday interrupting your precious sleep. So in short, don’t rely on single points of failure.

A single node would also be inelastic. Most applications get a lot of traffic at some points of the day and get little traffic at other times. This means that if you are an engineer at a streaming service company on which people watch shows and movies, chances are you will get more traffic on Friday evenings than, say, 3:00 PM on a Monday. If you require a node with 16 Gigs of RAM on Friday evenings, you are probably wasting it on a Monday.

If you had the option of smaller multiple nodes, however, you could add nodes when demand rises and remove them when demand decreases.

### Low write performance

Cassandra really shines when it comes to writing performance.

Its distributed nature, and generally, asynchronous writes to peer nodes translate into very quick writes regardless of the size of data in the system.

This is unlike most database systems where writes end up being slower as you add more data into the system. This can be due to a number of factors like locks or compromising on performance for ACID.

### Nonlinear scaling

Most databases scale in a non-linear manner. This means that doubling the nodes doesn’t often mean doubling the performance. Doubling the nodes would generally mean _less_ than double the performance.

![[Pasted image 20230222184141.png]]

This is because performance in traditional databases also depends on how much data they hold, and not just the operation you want to do. So Postgres databases may need to do a lot of work for inserting a single row (such as validating the unique keys in the row, ensuring foreign mappings, ensuring strong consistency, etc.) if it holds ten million rows, vs when it holds just ten rows.

This is again boiling down to how complex the relational model really is. There is a lot of overhead for any operation you do in a traditional database.

This is not true for Cassandra.

Cassandra scales linearly. This means that doubling the nodes in the system or doubling the computing power in the system would double the performance of the system.

![[Pasted image 20230222184236.png]] https://netflixtechblog.com/benchmarking-cassandra-scalability-on-aws-over-a-million-writes-per-second-39f45f066c9e

You can see that writes/s in Cassandra scale linearly by node count.

This is awesome if you think your application is going to get a lot of writes per second in the future.

## Why should you not use Cassandra?

### When you want a lot of different types of queries or you can’t predict your data usage

While Cassandra has a lot of strong points, it’s far from perfect. It comes with its own set of headaches. If it didn’t, we’d all be using Cassandra all the time.

One of these headaches is its data modelling. Data modelling in Cassandra is not simple. In fact, this trait is shared by many other databases that were inspired by the famous Dynamo paper. In short, queries that you are allowed to execute depending on your data model. And even if you are allowed to execute queries, its performance depends on the decisions you made when deciding the data model for the table.

Data modelling in Cassandra is a complex topic which I’d leave for another day, but know that you need to know which queries you will execute _before_ you model your data.

### When a single node works

Distributed systems are complex, and often require a lot more plumbing than a single node system.

So, it’s pretty simple, if a single node database works for you, then Cassandra probably doesn’t bring much to the table.

If you just want more read performance and a single node database that can handle the writes, even then chances are Cassandra isn’t the perfect fit. Cassandra’s true use case, in a single word, is scale. Scale at the level of thousands of RPS, at the level of hundreds of gigabytes of RAM.

So, as I said before, don’t use a Lamborghini in city traffic, you’d just end up wasting the gas.

### When you want strong ACID compliance

Cassandra(and in fact, most of the Dynamo-style databases) make trade-offs with ACID. All of them, to a certain extent, offer higher performance, and easier scaling, but compromise a bit on atomicity, consistency, and isolation.

It’s wrong to say Cassandra doesn’t offer any isolation or consistency or atomicity. It’s a little bit more nuanced than that. To avoid diving into it right now, know that it supports ACID transactions to some extent, but not as strictly as relational databases do. For example, you’d get partition-level atomicity(although without rollback) and you’d get strong consistency if you really want to(but nobody wants to use it because you’d compromise heavily on availability). So think of it as Cassandra supports ACID but terms and conditions apply.

If you want ACID compliance, you should probably look somewhere else.

### When you want many-to-many mappings or join tables

Cassandra doesn’t support a relational schema with foreign keys and join tables. So if you want to write a lot of complex join queries, then Cassandra might not be the right database for you.

### When you don’t want a rigid schema

If you think individual items in your table should be flexible and have different columns, then perhaps you should look at some other database like a document database like Mongo.

## Conclusion

In my (probably controversial) opinion, Cassandra is not a pleasant database to work with. I’d rather work with Postgres or MySQL or something else which gives me guarantees for consistency, good transactional support and a lot more. For a lot of applications, however, these kinds of databases aren’t practical at the scale they receive.

Cassandra and similar databases are more barebones than traditional relational databases, but they are built for scale. They compromise on a lot of features for performance _but they do deliver performance_.

My initial analogy of comparing Cassandra to a fast supercar still holds true in my opinion. Think of Postgres as the Rolls Royce Phantom, a super luxurious car that will give you all the bells and whistles money can buy, but when you want to floor the gas and you want absolute raw performance, a Lamborghini Huracan will beat the Phantom hands down.

#distill

----

## References
1. https://medium.com/geekculture/system-design-solutions-when-to-use-cassandra-and-when-not-to-496ba51ef07a