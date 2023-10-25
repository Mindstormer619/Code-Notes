---
created: 2023-08-25 18:19
alias: 
---
#### Summary
+ 

----
# Need for Data-Intensive thinking

Why design applications by focusing on [[Data Systems]], or data specifically? It's because many applications today are _data-intensive_ as opposed to _compute-intensive._ The limiting factor is the amount and complexity of data, and the speed at which it is changing.

*Primitive* (or building block) data systems have various examples:
+ Databases: storing data
+ Caches: Remembering the result of an expensive operation
+ Search index: Searching/filtering data by keywords etc.
+ Stream processing: Handling asynchronous messages from one process to another
+ Batch processing: Periodically crunch a large amount of accumulated data.

The main concerns for data-intensive software systems are:
+ **[[Reliability]]:** The system works correctly even in the face of adversity.
+ **[[Scalability]]:** As the system grows, there must be reasonable ways of dealing with that growth.
+ **[[Maintainability]]:** Over time, different people will work with the system, and they should all be able to work on it productively, either to adapt the system or maintain current behavior.

----

## References
+ <% tp.file.cursor(2) %>