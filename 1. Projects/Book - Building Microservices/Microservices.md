---
created: 2023-08-11 17:40
modified: 2023-08-11 17:40
alias: 
---
#### Summary
+ 

----
# Microservices

Independently releasable services that are modeled around a business domain. The way a complex system is made is by connecting various services across networks -- the services are used as building blocks.

They are a _type_ of [[Service Oriented Architecture|SOA]], though one that is opinionated about how service boundaries should be drawn, and independent deployability is key. 

![[Pasted image 20230811175923.png]]

Internal implementation details are entirely hidden -- outside consumers only access data via restricted endpoints (usually over the network). They mostly avoid the use of shared DBs. Each microservice uses its own DB.

We follow the concept of [[Information Hiding]] in microservices. It's _essential_ to have clear, stable service boundaries -- these don't change frequently over time, and allow independent release.

## [[Service Oriented Architecture|SOA]] vs Microservices

In general, SOA can have coupled dependencies. This would not make them _microservices_, because they're not independently deployable. Think of microservices as a _specific approach_ to SOA (in the same way that [[Extreme Programming]] or [[Scrum]] are specific approaches to [[Agile]] software development).

## Key Concepts

### [[Independent Deployability]]

The idea that we can make a change to a microservice, deploy it, and release that change to our users, without having to deploy any other microservices. See the [[Independent Deployability|main page]] for more.

This is quite literally the most important concept in microservices.

### Modeled around a Business Domain

Techniques like [[Domain Driven Design]] can be used to structure code to better represent the real-world domain that software operates in. With microservices, we use the same idea for service boundaries.

Model your services around business domains, to make it easier to roll out new functionality and recombine microservices in different ways. The whole idea is to make it _rare_ to have to make changes to multiple microservices at once -- it takes a ton more work than making the same kind of change in a [[Monolith]]. 

Don't use a [[Layered Architecture]]. Often, changes in functionality span across layers (for a new feature you need to update the DB, backend and frontend all together). Make the services end-to-end slices of business functionality. The decision is to prioritize high cohesion of business functionality over high cohesion of technical functionality.

### Owning their own state

Generally avoid the use of shared DBs. If a microservice wants to access data held by another microservice, go and ask the second one for its data.

This allows clear functionality separation, between what we can change freely (internal representation) vs changing rarely (external contract).

We need to limit backward-incompatible changes. Breaking compatibility with consumers will force them to change as well. Hiding this internal state is analogous to the practice of [[Encapsulation]] in [[Object Oriented Programming]]. It's an example of [[Information Hiding]] in action.

> 💡 **Sharing DBs.** Don't share DBs unless you really need to -- and even then do everything you can to avoid it. Generally one of the worst things to do for [[Independent Deployability]].

### Size

The idea is to keep it as small as possible. Keep it to the size at which it can be easily understood.

> "A microservice should be as big as my (your) head."
> ~ James Lewis, technical director @ ThoughtWorks.

Different people's ability to understand something isn't always the same, and so this is a judgement call to make.

> have ... "as small an interface as possible."
> ~ Chris Richardson, author of _Microservice Patterns_.

The concept of size is highly contextual. Sam Newman urges people not to worry about it. Focus on these key areas:
+ How many microservices can you handle?
+ How do you define microservice boundaries to get the most out of them, without everything becoming horribly coupled?

### Flexibility

> "Microservices buy you options."
> ~ James Lewis, technical director @ ThoughtWorks.

^bae16d

They _buy_ you _options_ -- it means they have a cost, and you must decide if the cost is worth the options. The resulting flexibility can be on organizational, technical, scale, or robustness axes. ^82dcef

Adopting microservices is not binary, it's more like turning a dial. As you turn up the dial, you have more flexibility but also more pain points. Strongly advocate incremental adoption, and assess the impact as you go.

### Alignment of Architecture and Organization

The biggest reason we see [[Three Tiered Architecture]] so commonly in fullstack services is because it is based on how we organize our teams. See [[Conway's Law]] for more.

3-tier architectures are a good example of the law in action. In the past, the primary way IT orgs grouped people were in terms of their core competency: a DB engineer/admin team, Java dev team, and frontend developer team. Therefore the software architecture models the org structure.

The forces of optimization have changed. We now group people in [[Cross-Functional Teams]] to reduce handoffs and silos. We want to ship software quicker. So we organize teams in terms of the way we break our systems apart.

Most changes we make relate to changes in business functionality. Therefore, it is better to group code (and teams) according to cohesion of business function rather than technology. Each service may end up containing a mix of all layers, but that's a local implementation concern.

> 💭 Example: A single microservice owned by the Profile Team can expose a UI to allow customers to update their info, with the state of the customer stored within this microservice. Available genres are fetched by a `Catalog` microservice, and a `Recommendation` microservice gives favorite genre information.
> 
> ![[Pasted image 20230811202355.png]]
> 
> The `Customer` microservice encapsulates a _thin slice_ of each of the three tiers.

[[Cross-Functional Teams]] own an end-to-end slice of user facing functionality.

----

## References
+ [[1. Projects/Book - Building Microservices/README|Book - Building Microservices]]