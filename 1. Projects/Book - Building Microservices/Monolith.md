---
created: 2023-08-16 10:51
alias: 
---
#### Summary
+ 

----
# Monolith

Monolithic architectures are where all the functionality in a system must be deployed together. Contrast this with [[Microservices]], where the idea is that each service has [[Independent Deployability]]. 

## Single Process Monolith

Probably the simplest idea of a monolith -- all the code is packed into a single process. There might be multiple *instances* for robustness or scaling, but the instances are completely independent and each contains the entire code for the application. They usually read or store data from a DB, which results in the following architecture:
![[Single process monolith.canvas|Single process monolith]]
[DHH made the case effectively](https://m.signalvnoise.com/the-majestic-monolith/) that such an architecture makes sense for smaller organizations.

## Modular Monolith

It's a subset of the [[#Single Process Monolith]]. Here the single process consists of separate modules, with well defined module boundaries for high degree of parallel work. However, the modules all still need to be combined together for deployment.

It's simpler to deploy than a [[Microservices|microservice]], but the downside is that code modules are loosely coupled, but the DB tends to lack the same decomposition. This makes it very difficult to pull the monolith apart in the future.

> ☝ Shopify is an example of an organization that uses this well.

#todo [[2023-08-16]]




----

## References
+ <% tp.file.cursor(2) %>