# Designing Data Intensive Applications - Storage And Retrieval
> Created: 2022-11-06 16:32
> #book 

## DB Data Structures

Fastest writes are through a simple append-only log. Simply add new lines to a log file. The problem is that reading from this file becomes an O(n) operation, which is terrible for large numbers of records. Regardless, many real databases do use an append-only log as a record of certain types of operations and data.

> 💡 **Indexes.** An 

----

## References
1. [Designing Data Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) Chapter 3