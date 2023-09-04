---
created: 2023-08-11 18:01
modified: 2023-08-11 18:01
alias: 
---
#### Summary
+ 

----
# Information Hiding

System design principle: Hiding as much information as possible inside a component and exposing as little as possible via external interfaces. This allows for clear separation between what can easily change and what is more difficult to change.

Relevant to [[Microservices]], where changes inside a microservice boundary should not affect upstream consumers, enabling independent releasability of functionality. It allows for looser [[Coupling]] and stronger [[Cohesion]].

The [[Hexagonal Architecture]] pattern describes the importance of keeping the internal implementation separate from its external interfaces. 

Also see: [[Encapsulation]]

----

## References
+ [[1. Projects/Book - Building Microservices/README|Book - Building Microservices]]