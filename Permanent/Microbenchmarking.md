# Microbenchmarking
> Created: 2022-05-28 10:42

**Microbenchmarks** are tests designed to measure the performance of a very small unit of code. E.g. using a synchronized method vs an unsynchronized version.

## Pitfalls

With microbenchmarks, the biggest issue is ending up measuring the performance of something that's not actually your code. It's easy to fall into the trap of thinking that one piece of code performs worse than another, but that might easily be because what's being measured doesn't reflect real-world performance.



----

## References
1. [[Java Performance - An Approach To Performance Testing]]