---
created: 2023-08-11 18:06
modified: 2023-08-11 18:06
alias: SOA
---
#### Summary
+ 

----
# Service Oriented Architecture

Design approach in which multiple services collaborate to provide a certain end set of capabilities. Typically, _service_ refers to a completely separate OS process. Communication between services occurs via calls across a network rather than method calls within a process.

SOA emerged to combat challenges of large, [[Monolith|monolithic]] applications. It promotes reusability of software, and to make it easier to maintain/rewrite software.

The problem is that there's a general lack of consensus on how to do SOA _well_. Many of the problems laid at the door of SOA are actually problems with things like communication protocols (e.g., SOAP), vendor middleware, a lack of guidance about service granularity, or the wrong guidance on picking places to split your system. A cynic might suggest that vendors co-opted (and in some cases drove) the SOA movement as a way to sell more products, and those selfsame products in the end undermined the goal of SOA.

----

## References
+ [[1. Projects/Book - Building Microservices/README|Book - Building Microservices]]