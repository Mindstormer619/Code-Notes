---
created: 2023-08-21 16:29
alias: 
---
#### Summary
+ 

----
# Common Mistakes with [[MongoDB]]

In hopes of making it easier for other people, here is a list of common mistakes.

## Creating a MongoDB server without authentication

MongoDB installs without authentication by default. This is fine on a workstation, accessed only locally. But because MongoDB is a multiuser system that likes to use as much memory as it can, it is much better installed on a server, loaded up to the hilt with RAM, even for development work. To install it on a server on the default port without authentication is asking for trouble, especially when one can execute arbitrary JavaScript within a query.

There are several authentication methods, but user ID/password credentials are easy to install and manage. Do that method while you think about your [fancy LDAP-based authentication](https://docs.mongodb.com/manual/core/security-ldap-external/). Use a different port to the default.

## Forgetting to tie down MongoDB’s attack surface

MongoDB’s [security checklist](https://docs.mongodb.com/manual/administration/security-checklist/) gives good advice on reducing the risk of penetration of the network and of a data breach. Unless there is a very good reason to use  `mapReduce`, `group`, or `$where`, you should [disable the use of arbitrary JavaScript](https://lockmedown.com/securing-node-js-mongodb-security-injection-attacks/) by setting `javascriptEnabled:false` in the config file. Also, standard MongoDB is not encrypted -- it is also wise to [Run MongoDB with a Dedicated User](https://docs.mongodb.com/manual/administration/security-checklist/#run-mongodb-with-a-dedicated-user) with full access to the data files restricted to that user so as to use the operating systems own file-access controls.

## Failing to design a schema

MongoDB doesn’t enforce a schema. This is not the same thing as saying that it doesn’t need one. If you really want to save documents with no consistent schema, you can store them very quickly and easily but retrieval can be the very devil. (See [[Schema-on-read]], [[Comparison of data models]])

## Forgetting about collations (sort order)

This can result in more frustration and wasted time than any other misconfiguration. MongoDB defaults to [using binary collation](https://jira.mongodb.org/browse/SERVER-1920). When you create a MongoDB database, use an [accent-insensitive, case-insensitive](https://weblogs.sqlteam.com/dang/archive/2009/07/26/Collation-Hell-Part-1.aspx) collation appropriate to the [languages and culture of the users](https://derickrethans.nl/mongodb-collation-revised.html) of the system. This makes searches through string data so much easier.

## Creating collections with large documents

MongoDB is happy to accommodate large documents of up to 16 MB in collections. Because large documents can be accommodated doesn’t mean that it is a good idea. MongoDB works best if you keep individual documents to a few kilobytes in size, treating them more like rows in a wide SQL table. Large documents will cause [several performance problems](https://www.reddit.com/r/mongodb/comments/573fqr/question_mongodb_terrible_performance_for_a/).

## Creating documents with large arrays

It is best to keep the number of array elements well below four figures. If the array is added to frequently, it will outgrow the containing document so that [its location on disk has to be moved](http://docs.mongodb.org/manual/core/data-model-operations/#document-growth), which in turn means [every index must be updated](http://docs.mongodb.org/manual/core/write-performance/#document-growth). A lot of index rewriting is going to take place when a document with a large array is re-indexed, because there is a separate [index entry for every array element](http://docs.mongodb.org/manual/core/index-multikey/). This re-indexing also happens when such a document is inserted or deleted.

You might think that you could get around this by not indexing arrays. Unfortunately, without the indexes, you can run into other problems. Because documents are scanned from start to end, it takes longer to find elements towards the end of an array, and [most operations dealing with such a document would be slow](http://grokbase.com/t/gg/mongodb-user/128r0h5gzw/inserting-into-300-000-size-embedded-array-is-slow-even-w-o-indexes).

## Forgetting that the order of stages in an aggregation matters

For example, you need to make sure that the data is reduced as early as possible in the pipeline via `$match` and `$project`, sorts happen only once the data is reduced, and that lookups happen in the order you intend. Having a query optimizer that removes unnecessary work, orders the stages optimally, and chooses the type of join can spoil you. MongoDB gives you more control, but at a cost in convenience.

## Using fast writes

Never set MongoDB for high-speed writes with low durability. This ‘file-and-forget’ mode makes writes appear to be fast because your command returns before actually writing anything. If the system crashes before the data is written to disk, it is lost and risks being in an inconsistent state. Fortunately, 64-bit MongoDB has journaling enabled.

Journaling will ensure that the database is in a consistent state when it recovers and will save all the data up to the point that the journal is written. The duration between journal writes is configurable using the [`commitIntervalMs`](https://docs.mongodb.com/manual/reference/configuration-options/#storage.journal.commitIntervalMs) run-time option.

To be confident of your writes, make sure that journaling is [enabled (`storage.journal.enabled`) in the configuration file](https://docs.mongodb.com/manual/reference/configuration-options/#configuration-file) and the commit interval corresponds with what you can afford to lose.

## Sorting without an index

In searches and aggregations, you will often want to sort your data. Hopefully, it is done in one of the final stages, after filtering the result, to reduce the amount of data being sorted. Even then, [you will need an index that can cover the sort](https://studio3t.com/knowledge-base/articles/mongodb-index-strategy/). Either a single or compound index will do this.

----

## References
+ https://www.infoq.com/articles/Starting-With-MongoDB/