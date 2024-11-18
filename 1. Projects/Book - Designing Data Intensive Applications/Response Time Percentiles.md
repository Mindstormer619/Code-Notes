---
created: 2023-02-13 16:50
aliases:
  - Latency Percentiles
  - p90
---
> [!summary]-
> + 

# Response Time Percentiles

![[Pasted image 20220720123608.png]]

Response time is generally considered as a distribution of values, not a single one. This is expressed as a set of *percentile* measures.

**p50.** The 50th percentile, or the median. Half the user requests are served within this time, the other half take longer than this.

**p90, p99, p999.** Outlier, or tail-latency percentiles. p99 = 99th percentile, while p999 is 99.9th percentile. If p99 = 1.5s, 1% of the requests take longer than 1.5 seconds to complete.

**The high percentiles are called _tail latencies_** and they matter because they directly affect user experience of the service. **The customers with the slowest requests will also often have the most data on their accounts**, because they have made the most purchases -- i.e. they are the most valuable customers. However, there are diminishing returns (e.g. p999 may matter for your product, but not p9999).

**We don't generally use arithmetic means as a response time metric, because it doesn't tell you how many users experienced that delay**. For example, if the mean response time is 1 second, it could mean that 4 out of every 5 users has a 0.1s response, but every fifth user has an atrocious 4.6s response time. This information is completely hidden (however in percentile measures, p50 will be 0.1s, but p90 will reveal the 4.6s time).

Percentiles are often used in [[Service Level Objectives]] and Agreements.

## Head-of-line Blocking

It is important to measure response times at the client, because head-of-line blocking can occur. This is when queueing delays occur at the server, where a few slow requests slow down the processing of every request that comes after them, including ones that would normally be processed faster.

> ☝ Head-of-line blocking is an [issue faced](https://engineering.cred.club/head-of-line-hol-blocking-in-http-1-and-http-2-50b24e9e3372) by HTTP/1 and HTTP/2.

## Tail Latency Amplification

If a user request requires multiple **parallel backend calls** to complete the operation, **the operation will be as slow as the slowest call in the set**. Therefore, even if only a small percentage of the calls are slow, the chance of a slow request for the end user increases. 

![[Pasted image 20230213172053.png]]

## Measuring Response Time

Efficient calculation of response time needs to be done by ongoing monitoring. A **rolling window** of response times over, say, last 10 minutes can be used (calculate median and percentiles over the window and use that in graph).

Sorting the list every time may be inefficient. There are efficient approximations at minimum CPU/memory like **forward decay, t-digest, HdrHistogram**. **Averaging percentiles is mathematically meaningless** — the right aggregation is to add the histograms.

----

## References
1. [[Designing Data Intensive Applications - Reliable, Scalable, Maintainable Apps#Performance]]