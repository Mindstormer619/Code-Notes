# Steps for System Design Interviews
> Created: 2023-02-22 17:13
> #book 

Steps for an SDI:

1. **Requirements Clarification.** Design questions are open ended and do not have one correct answer. Start with clarifying requirements. For example (for Twitter):
	+ Will users be able to post tweets and follow other people?
	+ Will tweets contain photos / videos?
	+ Do we need to display trending topics?
	+ Do we need push notifications for new tweets?
2. **System interface definition.** Define the APIs required by the system. Helps to determine the exact contract. For example: `postTweet(userId, tweetData, tweetLocation, ...)`
3. **Back of the envelope estimation.** For scale estimation and narrowing down on solutions.
	+ What scale is expected from the system?
	+ How much storage will we need?
	+ What network bandwidth are we expecting?
4. **Defining data model.** Clarify how data is structed, how it will flow/partition. Data storage, transportation, encryption etc. SQL or NoSQL? What kind of block storage, if any?
5. **High-level design.** Draw a block diagram with boxes representing the core components of the system.
6. **Detailed design.** Dig deeper into two or three of the high-level components (guided by the interviewer). There is no single answer, just give options considering tradeoffs and constraints of the system.
	+ Since we are storing massive data, how do we partition it? Should we try to store all data in a single DB? What issues could that cause?
	+ How do we handle hot users?
	+ How do we optimize to scan for the latest tweets?
	+ How much do we cache? Which layer do we introduce caching?
	+ Which components need better load balancing?
7. **Identifying and resolving bottlenecks.** 
	+ Is there a SPOF (single point of failure)? How to mitigate?
	+ Do we have enough replicas to prevent data loss?
	+ Availability on replica failure?
	+ How are we monitoring system performance and health?


----

## References
1. Grokking The System Design Interview