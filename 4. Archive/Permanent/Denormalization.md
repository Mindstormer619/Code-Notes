# Denormalization
> Created: 2023-05-24 13:04
> **Also see: [[Data Normalization]]**

Denormalization violates the principles of [[Data Normalization]] by adding redundant data to a DB in order to improve performance.

The basic idea is to store data in multiple places or a simpler format, for easy retrieval.

## Types
+ Data duplication. Storing the same data in multiple places, e.g. a DB might store the customer's address in both the `customers` and `orders` table.
+ Data summarization. Storing *aggregated data* in a single place. E.g. the DB might store the _total number of orders_ placed by each customer in a single table, so that you don't have to re-summarize it on every read (the summary can be updated periodically or on writes).

## Advantages
+ Simpler schema. Instead of having a complex web of interconnected fields for, say an address, you can have a single text field which is more flexible and easy to update.
+ Higher read performance. In the above example on data duplication, to get the address of an order, you can read it directly off the `orders` tuple rather than have to do a join to the `customers` table.

## Drawbacks
+ Increased data storage. The duplicates can increase the amount of data you need to store.
+ Reduced data integrity. This is the big one -- you have to now manage the consistency of all the copies of data over your DB yourself. Changes to the data in one place need to be replicated to other places where the data is stored.
+ Increased complexity. Can make the DB more difficult to debug / understand, because you have multiple places where a single piece of data can live.

## When to use?
You should only use denormalization if the benefits outweigh the drawbacks. Generally speaking, high read performance requirements are needed if you should consider denormalizing.

----

## References
1. [[2. Areas/Databases/Denormalization]]