---
created: 2023-08-21 16:29
alias: 
---
#### Summary
+ 

----
# Common Mistakes with MongoDB

In hopes of making it easier for other people, here is a list of common mistakes.

## Creating a MongoDB server without authentication

MongoDB installs without authentication by default. This is fine on a workstation, accessed only locally. But because MongoDB is a multiuser system that likes to use as much memory as it can, it is much better installed on a server, loaded up to the hilt with RAM, even for development work. To install it on a server on the default port without authentication is asking for trouble, especially when one can execute arbitrary JavaScript within a query.

There are several authentication methods, but user ID/password credentials are easy to install and manage. Do that method while you think about your [fancy LDAP-based authentication](https://docs.mongodb.com/manual/core/security-ldap-external/). Use a different port to the default.

## Forgetting to tie down MongoDB’s attack surface

MongoDB’s [security checklist](https://docs.mongodb.com/manual/administration/security-checklist/) gives good advice on reducing the risk of penetration of the network and of a data breach. Unless there is a very good reason to use  `mapReduce`, `group`, or `$where`, you should [disable the use of arbitrary JavaScript](https://lockmedown.com/securing-node-js-mongodb-security-injection-attacks/) by setting `javascriptEnabled:false` in the config file. Also, standard MongoDB is not encrypted -- it is also wise to [Run MongoDB with a Dedicated User](https://docs.mongodb.com/manual/administration/security-checklist/#run-mongodb-with-a-dedicated-user) with full access to the data files restricted to that user so as to use the operating systems own file-access controls.

## Failing to design a schema

MongoDB doesn’t enforce a schema. This is not the same thing as saying that it doesn’t need one. If you really want to save documents with no consistent schema, you can store them very quickly and easily but retrieval can be the very devil. (See [[Schema-on-read]], [[Comparison of data models]])

## Forgetting about collations (sort order)

This can result in more frustration and wasted time than any other misconfiguration. MongoDB defaults to [using binary collation](https://jira.mongodb.org/browse/SERVER-1920).

----

## References
+ https://www.infoq.com/articles/Starting-With-MongoDB/