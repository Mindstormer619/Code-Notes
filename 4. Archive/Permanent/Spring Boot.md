# Spring Boot
> Created: 2023-05-31 20:27
> #todo Reformat

Function -- to remove boilerplate

+ configuration management
	+ load configs from files
+ req/response handling, singletons
	+ DI scoping
+ inversion of control
+ annotation-driven web routing (Spring Web)
	+ and other shit
	+ authentication, authorization, all the other middleware jazz
+ HTTP client library
	+ JSON serdes included
+ ORM
	+ SpringData
	+ internally uses Hibernate (can be configured)
	+ automatically figures out SQL flavors from your DB dependency
	+ Support for Cassandra and MongoDB as well
+ thread pool management
+ async job scheduling
+ cache management

Spring Boot: smart defaults for configurations on Spring

## Spring vs Spring Boot
https://www.baeldung.com/spring-vs-spring-boot

Spring generally requires some setup (like min dependencies for a web project are `spring-web` and `spring-webmvc`) but Spring Boot does opinionated startup.

----

## References
1. Chaitanya
2. https://www.baeldung.com/spring-vs-spring-boot