---
created: 2023-08-21 16:29
alias: 
---
#### Summary
+ 

----
# Common Mistakes with MongoDB

In hopes of making it easier for other people, here is a list of common mistakes.

### Creating a MongoDB server without authentication

MongoDB installs without authentication by default. This is fine on a workstation, accessed only locally. But because MongoDB is a multiuser system that likes to use as much memory as it can, it is much better installed on a server, loaded up to the hilt with RAM, even for development work. To install it on a server on the default port without authentication is asking for trouble, especially when one can execute arbitrary JavaScript within a query.



----

## References
+ https://www.infoq.com/articles/Starting-With-MongoDB/