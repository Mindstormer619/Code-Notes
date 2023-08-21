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

## Distributed Monolith

Consists of multiple services, but for whatever reason, they all need to be deployed together. DON'T DO THIS. 

> "A distributed system is one in which the failure of a computer you didn't even know existed can render your own computer unusable."
> ~ Leslie Lamport

It has all the disadvantages of both monoliths and a distributed system. It usually results in environments where not enough focus was placed on information hiding and cohesion of business functionality. The highly coupled architecture causes changes to ripple across service boundaries.

## Con: Delivery Contention

Delivery contention results when different developers want to change the same piece of code, or different teams want to push functionality at different times. Since the entire monolith needs to be deployed together, this can cause issues in delivery.

There is a general confusion along the lines of who owns what (somewhat mitigated in [[#Modular Monolith]] models). This is generally a much bigger problem in monoliths than [[Microservices]].

> ☝ Consider the issues we had in GS with deployment queues (_queues_ is a strong word, it was basically first-come-first-served fights because there was no _automated_ queue to place your build on) on Thursday -- builds took over an hour to run, and if you didn't "reserve" your build right when the previous build ended, you could not run your build.

## Pros of Monoliths




----

## References
+ <% tp.file.cursor(2) %>