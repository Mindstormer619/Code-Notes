---
created: 2023-08-21 16:24
alias: 
---
#### Summary
+ 

----
# MongoDB

MongoDB is a [[Document Databases|document DB]] which organizes data in the form of _documents_, which are data structures with field and value pairs. MongoDB uses BSON (Binary JSON) to store documents.

![A MongoDB document.](https://www.mongodb.com/docs/manual/images/crud-annotated-document.bakedsvg.svg)

Advantages of documents:
+ Correspond to native data types in languages.
+ Embedded docs and arrays reduce the need for expensive joins.
+ Dynamic schema supports fluent polymorphism.

As with any dynamic schema DB, MongoDB is [[Schema-on-read]] and not [[Schema-on-write]].

MongoDB supports:
+ Collections: groups of documents (think of tables in relational models) 
+ Read-only views
+ On-demand materialized views.

## Features
+ High performance data persistence, with embedded data models and complex indexes
+ Query API for CRUD, data aggregation (see [[MongoDB Aggregation Pipelines]]), text search and geospatial queries.
+ High availability via a [[MongoDB replica set|replica set]] providing automatic failover and data redundancy.
+ Horizontal scalability with [[Sharding]] for distributing data across a cluster of machines, and zones of data based on the shard key.
+ Multiple storage engines: WiredTiger and In-Memory Storage (along with pluggable 3rd party engines).

----

## References
+ https://www.mongodb.com/docs/manual/introduction/
+ The Little MongoDB Book ~ Karl Seguin
+ Awesome List at https://github.com/ramnes/awesome-mongodb#resources