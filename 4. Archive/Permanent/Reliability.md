# Reliability
> Created: 2023-02-01 15:37

**Reliability** is the idea that something _works correctly, even when things go wrong_. The system works correctly even in the face of adversity. Reliability is generally not optional (except in cases of prototypes, where reliability sacrifices should be stated explicitly).

Things can and *will* go wrong. Things that go wrong in systems are called [[Faults]]. A [[Failures|Failure]] is when a [[Faults|Fault]] causes the system as a whole to stop providing the required service to the user. We design systems to be fault-tolerant, which means that faults should not cause failures.



----

## References
1. [[Designing Data Intensive Applications - Reliable, Scalable, Maintainable Apps]]