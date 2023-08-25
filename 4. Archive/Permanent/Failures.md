# Failures
> Created: 2023-02-01 16:06

A [[Faults|fault]] is when one component of a system deviates from its spec. It escalates into a failure when the system as a whole stops providing the required service to the user.

The idea of designing **fault-tolerant** systems is to ensure that faults do not cause failures, as faults are generally unavoidable. This practice is called [[Reliability]].

It is important to understand the parameters of likelihood of different faults, and to calculate whether or not that likelihood needs to be guarded against, given the *cost* of making the system tolerant of that fault. For instance, the fault described as an asteroid or black hole which impacts the planet, will render almost every single system offline. This failure can be prevented by keeping replicas of the system in space. But the cost is generally prohibitive, and the likelihood of the fault is low enough to not justify the cost in guarding against it.

In fault-tolerant systems, it can make sense to _increase_ the rate of faults by triggering them deliberately. The objective is to make sure that the faults don't cause failure of the system.
> ☝ Netflix's [Chaos Monkey](https://github.com/Netflix/chaosmonkey) is an example -- it randomly terminates VM instances and containers in prod, to expose engineers to faults more frequently. 

----

## References
1. [[1. Projects/Book - Designing Data Intensive Applications/README|DDIA]]