# Insights on Caching
> Created: 2022-12-09 11:45

The idea with caching efficiently is to cache for whatever data _takes the most time_. Now this might sound intuitive, but the reasoning done practically is often flawed.

For example, consider a system relying on data coming from an external system. Let us explore two separate scenarios:
1. The data is large in size, and fetching the data takes longer the larger the data gets. In this case, the bottleneck to performance is the speed of data reads in the external system.
2. The data supplied by the external system is relatively small in size, but takes time through the network getting to our system. In this case, the bottleneck is not the data read, but rather the request RTT across the network.



----

## References
1. [[Caching Pitfalls]]