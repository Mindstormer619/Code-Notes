# Faults
> Created: 2023-02-01 15:47

A **fault** is when one component of a system deviates from the spec. Faults which cause [[Failures]] compromise the [[Reliability]] of the system. 

Faults can be unavoidable. For example, pretty much any system that isn't hosted with replication in space will die if the earth gets swallowed by a black hole. Hence, we design systems to be **fault-tolerant**, where we ensure that faults do not cause system failures.

Types of faults are listed below:

+ **Hardware Faults**
	+ Hardware fails all the time. Hard disks have an [[MTTF]] of **10-50 years**. Hence on a cluster of 10,000 disks, one disk dies per day on average.
	+ We add **redundancy** to the system — backup/[[RAID]] configs for the disks, dual power supply, hot-swappable CPUs, batteries for power backup etc.
	+ At massive scale, hardware component redundancy might not be enough, and we might need to use software-controlled fault tolerance (e.g. to spin up additional machines for new VMs).
+ **Software Errors**
	+ Software bugs generally show strongly correlated issues, which can take out entire instances of an application at once.
	+ The bug is usually because an assumption was made somewhere which held true, until it wasn't true anymore.
+ **Human Errors**
	+ Hardware faults play a role only in 10-25% of outages. The leading cause is config errors by human operators.
	+ How do we design for reliability despite unreliable humans?
		+ Design systems to minimize opportunity for error under normal operation, without making the system too restrictive.
		+ Decouple the places where people make mistakes from the places where the mistakes can cause failures — create *sandboxes*.
		+ Test thoroughly, especially with automated tests.
		+ Allow quick and easy recovery from human error via rollbacks, gradual rollouts, tools to recompute data etc.
		+ Setup detailed monitoring.
		+ Implement good management and training.


----

## References
1. [[Designing Data Intensive Applications - Reliable, Scalable, Maintainable Apps]]