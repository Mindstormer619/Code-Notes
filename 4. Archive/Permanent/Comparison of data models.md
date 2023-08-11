# Comparison of data models
> Created: 2023-05-24 12:52
> *This page talks about comparison between relational and nonrelational data models.*

+ **Simplicity of application code.**
	+ If the data in the app has a document-like structure where you load the whole tree at once, go with document models. Relational models **shred** data (split into multiple tables) and that makes your app code more complex.
	+ If you need to frequently reference nested items directly (without loading the whole structure into memory) then go with relational models.
+ **Data relationships / joins**
	+ Use a lot of many-many joins? Avoid document structures -- you will have to write the joins and handle failures yourself.
+ **Schema flexibility**
	+ Document DBs are conceptually **schema-on-read** (implicit structure of data, only interpreted when the data is read) as opposed to relational DBs which are **schema-on-write** (explicit schema, the DB ensures all written data conforms).
	+ `ALTER TABLE` on relational is generally fast (see Note for exception)
	+ With schema-on-read, the application code usually handles old schemas using an `if` block (or similar). Think of it as a dynamically typed language.
+ **Data locality**
	+ A document store keeps the whole document in contiguous memory blocks -- faster retrieval if you want the whole document.
	+ But if you want only bits of it, then it gets worse for perf, because the whole thing needs to be picked.
	+ Some relational DBs (e.g. Google Spanner) also give data interleaving options for locality on disk.

> ❗️ **Note:** MySQL creates a copy of the table before altering it, so schema alteration can take hours!

----

## References
1. [[Designing Data Intensive Applications - Data Models and Query Languages]]