# Scalability
> Created: 2023-02-13 16:40

The ability of the system to cope with increased [[Load]]. Scalable systems can be reconfigured -- either automatically or manually -- to deal with increases in load, which could come in the form of more concurrent connections, cache hits etc.

## Performance

**Throughput:** Records processed per unit time, or the total time taken to run a job on a dataset of a certain fixed size. This matters more for batch-processing systems.

**Response Time:** Time between a client sending a request and receiving a response. Matters more for *online* systems. We consider this as a [[Response Time Percentiles|distribution of values]], not a single one.

> 💡 **Latency vs Response Time.** Response time is what the client sees; it includes network delays, queueing delays etc. Latency is the duration that a request is waiting to be handled, during which it is _latent_, awaiting service.

## Coping With Load

**Scaling up.** Vertical scaling, i.e. moving to a more powerful machine.

**Scaling out.** Horizontal scaling, i.e. distributing the load across multiple smaller machines. This is also called a *shared-nothing architecture.*

It is not a dichotomy — a good architecture often involves a pragmatic mixture of approaches. High-end machines can get expensive individually, but sometimes it might be cheaper to get a few powerful machines than set-up the management for a large number of small virtual machines.

**Elastic scaling.** Automatically adding computing resources to cope with increased load. Contrasted with scaling manually, which is not good for frequent load unpredictability, but otherwise might be better because manual scaling is simpler to setup and has fewer operational surprises.

**Stateful systems.** Generally keep your databases on a single node (scale up) until scaling costs or other requirements force you to make it distributed. Scaling stateful systems can get hairy.


----

## References
1. [[Designing Data Intensive Applications - Reliable, Scalable, Maintainable Apps]]