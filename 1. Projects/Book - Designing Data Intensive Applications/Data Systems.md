---
created: 2023-08-25 18:28
alias: 
---
#### Summary
+ 

----
# Data Systems

Why lump tools like DB, message queues etc. as _data systems_ when they have different access patterns and uses?

One, a lot of new tools have multiple use cases, and no longer neatly fit into the traditional categories.
> ☝ Redis is both a message queue and a datastore
> ☝ [[Apache Kafka]] is a message queue which gives DB-like durability guarantees.

Two, many apps have wide-ranging requirements such that a single tool cannot meet all data processing / storage needs. Different tools are stitched together using app code.

![[Pasted image 20230825183200.png]]

The new special-purpose data system you create from the tools hides the implementation details via the API (see [[Information Hiding]]) and therefore forms a new composite.

----

## References
+ <% tp.file.cursor(2) %>