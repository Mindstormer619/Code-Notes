---
created: 2023-08-11 18:18
modified: 2023-08-11 18:18
alias: 
---
#### Summary
+ 

----
# Independent Deployability

The idea that we can make a change to a [[Microservices|microservice]], deploy it, and release that change to our users, without having to deploy any other microservices. This should _actually_ be the way by which deployments are managed by default in your system.

For this, microservices should be [[Coupling|loosely coupled]], we must be able to change one service without changing anything else. This is only possible with explicit, well-defined, stable contracts between services.

You can use this as a _forcing function_: by forcing this outcome, there are a number of ancillary benefits. See [[Extreme Programming]] for examples of the same kind of thing in software delivery.

## Tips

+ Have proper delineation of internal implementation from external service boundary definitions.
	+ This way, you can change internals without changing anything exposed to the outside.
+ Do not share DBs. Refer to [[Microservices#Owning their own state]] for more.



----

## References
+ [[1. Projects/Book - Building Microservices/README|Book - Building Microservices]]