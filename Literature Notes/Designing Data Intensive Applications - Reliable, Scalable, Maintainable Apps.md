# Designing Data Intensive Applications - Reliable, Scalable, Maintainable Apps
> Created: 2022-07-20 11:00
> #book 

## Data Systems

*Primitive* (or building block) data systems have various examples:
+ Databases: storing data
+ Caches: Remembering the result of an expensive operation
+ Search index: Searching/filtering data by keywords etc.
+ Stream processing: Handling asynchronous messages from one process to another
+ Batch processing: Periodically crunch a large amount of accumulated data

There may be other data system building blocks outside these.

When you combine several of these to make a service, you're hiding the implementation details from the users of that service, and have hence created a complex data system from these building blocks.

The main concerns for data-intensive software systems are:
+ **Reliability:** The system works correctly even in the face of adversity.
+ **Scalability:** As the system grows, there must be reasonable ways of dealing with that growth.
+ **Maintainability:** Over time, different people will work with the system, and they should all be able to work on it productively, either to adapt the system or maintain current behavior.

## Reliability

Typical expectations:
+ The application performs the expected function.
+ It can tolerate the user making mistakes / using the software in unexpected ways.
+ Performance is good enough for the required use case, under expected constraints.
+ It prevents any unauthorized access/use.

Therefore reliability roughly means: _continuing to work correctly, even when things go wrong_.

**Faults:** Things that can go wrong. We generally only care about certain types of faults. For e.g. any system will die if the earth gets swallowed by a black hole. It's important to explicitly state what kinds / levels of fault your system is designed to handle.

**Failure:** A _fault_ is when one component of the system deviates from its spec, while a _failure_ is when the system as a whole stops providing the required service to the user. It's generally impossible to reduce fault probability to 0, so we design _fault-tolerant_ mechanisms which prevent faults from causing failures.

Reliability is important in both critical and non-critical applications (users despise it if you irreversibly lose their personal data). In some cases like prototyping, you may sacrifice reliability, but do so _explicitly_.

### Hardware Faults

Hardware fails _all the time_. Hard disks are reported to have a Mean Time To Failure (MTTF) of 10-50 years. Therefore on a storage cluster of 10,000 disks, one will die per day on average.

**Redundancy:** You can add backup copies to reduce the failure rate of the system. e.g. RAID configs for disks, dual power supply, hot-swappable CPUs, batteries or diesel generators for backup power. 

Generally, hardware component redundancy is enough. However, at massive scale (e.g. AWS datacenters) you'll have a high probability of entire machine failures. A good example here is AWS virtual machine instances, where it's fairly common for the VM to become unavailable without warning. Hence you need systems that can tolerate the loss of entire machines by using software fault-tolerance.

### Software Errors

Generally hardware errors are not strongly correlated, and can mostly be considered independent. However, this is not the same for software errors. E.g.

+ A software bug that causes every instance of an app server to crash when given bad input -- Leap Second bug in Linux kernel in 2012.
+ Runaway process that uses up some shared resource
+ A service that the system depends on slows down / unresponsive / bugged.
+ Cascading failures: one fault triggers faults in other components.

These bugs usually show that the software _makes an assumption_ which was true, up until it wasn't.

### Human Errors

A study found that the leading cause of outages was configuration errors by operators. Hardware faults played a role in only 10-25% of outages.

Reliability despite unreliable humans? How?

+ Design systems in a way that minimizes opportunity for error. It's a tricky balance, because if you make the system too restrictive, people will start working around it and negate its benefit.
+ Decouple the places where people make mistakes from the places where they can cause failures. Provide _sandbox_ environments where people can play around and experiment safely without affecting prod.
+ Test thoroughly at all levels. Automated testing is very good for covering corner cases that rarely appear in normal operations.
+ Allow quick and easy recovery from human error. Make it fast to rollback config changes, roll out new code gradually, provide tools to recompute data.
+ Setup detailed and clear monitoring. This is also called **telemetry**.
+ Implement good management practices and training.

## Scalability

The ability for the system to cope with increased load. However, it is meaningless to say "X is scalable" -- it is important to describe the axis on which this load is measured.

### Load

Load is described with _load parameters_. E.g. requests per second for a webserver, ratio of reads to writes in a DB, simultaneously active user count in a chatroom, hit rate on a cache etc.

#### Case Study: Twitter

In Nov 2012, Twitter had these load parameters for their main operations:
+ Post Tweet: 4.6k rps average, 12k rps peak.
+ View Home Timeline: 300k rps.

12k writes per second is easy, but the issue is _fan-out_ which results from each user following many other users, and being followed by many people. There are two ways of implementing this operation:

1. Direct write but query read. Posting a tweet simply adds it to a tweet collection, but when a user requests their home TL, we query and put together the collection of recent tweets from all the users they follow:
   ```sql
   select tweets.*, users.* from tweets
	join users on tweets.sender_id = users.id
	join follows on follows.followee_id = users.id
	where follows.follower_id = @current_user
   ```
2. Direct read but query write. Maintain a cache for each user's home TL, and when a user _posts_ a tweet, go and update the cache for every one of their followers. The request to read the home TL is then cheap, because it has been computed ahead of time.

The first version of Twitter used Approach 1, but then they switched to Approach 2. This works better because the average rate of published tweets is almost two orders of magnitude lower than the rate of home TL reads. Therefore it is better to do more work at write time than read time.

Currently Twitter uses a hybrid of both approaches, where most users' tweets are fanned out to home timelines of their followers, but for a small number of users with a massive following (celebrities) are excepted from this fan-out.

### Performance

**Throughput:** The number of records processed per unit time, or total time it takes to run a job on a dataset of a certain size. This generally matters more for batch-processing systems.

**Response time:** Time between a client sending a request and receiving a response. Matters more for online systems. Should be considered as a _distribution_ of values, not a single one.

> **Latency vs Response Time.** Response time is what the client sees; it includes network delays, queueing delays etc. Latency is the duration that a request is waiting to be handled, during which it is _latent_, awaiting service.

![[Pasted image 20220720123608.png]]

Arithmetic mean isn't a very good metric, as it doesn't tell you how many users experienced that delay. Generally it is better to use _percentiles_.

The median is the 50th percentile, also called **p50**. Half the user requests are served within this time, the other half take longer than this.

To understand outliers, we see 95th, 99th, or 99.9th percentiles (**p95, p99, p999**). So if p99 = 1.5 seconds, 1% of requests take longer than 1.5 seconds to complete.

The high percentiles are called _tail latencies_ and they matter because they directly affect user experience of the service. The customers with the slowest requests will also often have the most data on their accounts, because they have made the most purchases -- i.e. they are the most valuable customers. However, there are diminishing returns (e.g. p999 may matter for your product, but not p9999).

----

## References
1. [Designing Data Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) Chapter 1