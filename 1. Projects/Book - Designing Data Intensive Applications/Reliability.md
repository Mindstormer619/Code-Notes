---
created: 2023-08-21 11:59
alias: 
---
#### Summary
+ 

----
# Reliability

Reliability is the idea that something _works correctly, even when things go wrong_. The system works correctly even in the face of adversity. Reliability is generally not optional (except in cases of prototypes, where reliability sacrifices should be stated explicitly).

Typical expectations:
+ The application performs the expected function.
+ It can tolerate the user making mistakes / using the software in unexpected ways.
+ Performance is good enough for the required use case, under expected constraints.
+ It prevents any unauthorized access/use.

Things can and *will* go wrong. Things that go wrong in systems are called [[Faults]]. A [[Failures|Failure]] is when a [[Faults|Fault]] causes the system as a whole to stop providing the required service to the user. We design systems to be fault-tolerant, which means that faults should not cause failures.

Reliability is highly important, even in most noncritical applications. While [[High Availability]] might not be critically needed, things like permanent data loss etc. can cause loss of user trust / reputation, lost productivity, and huge revenue loss costs. Be very conscious when sacrificing reliability for any reason.

----

## References
+ [[1. Projects/Book - Designing Data Intensive Applications/README|Book - Designing Data Intensive Applications]]