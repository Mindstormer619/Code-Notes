# Operability
> Created: 2023-02-13 19:43

> *Good operations can often work around the limitations of bad (or incomplete) software, but good software cannot run reliably with bad operations.*

An ops team is responsible for:
+ Monitoring the health of the system and restoring it if it goes down.
+ Tracking the cause for faults / failures.
+ Keeping software up to date.
+ Keeping tabs on how different systems affect each other.
+ Anticipating future problems and solving them before they occur (e.g. capacity planning).
+ Establishing good practices and tools
+ Performing complex maintenance tasks.
+ Maintaining the security of the system over various config changes
+ Defining processes that make operations predictable and stable.
+ Documenting system knowledge.

Data systems can make the work of ops teams easier:
+ Providing visibility into runtime behavior with good monitoring
+ Giving good support for automation and integration with tools
+ Avoiding dependency on individual machines (machines can be taken down for upgrade / maintenance)
+ Providing good documentation and operation models.
+ Providing good default behavior, with custom configurability
+ Self-healing when appropriate, but allowing manual control
+ Exhibiting predictable behaviors, avoiding surprises.


----

## References
1. [[Designing Data Intensive Applications - Reliable, Scalable, Maintainable Apps#Maintainability]]