# Java Performance - Introduction
> Created: 2022-05-22 20:57
> #book

## JVM Tuning Flags

+ **Boolean flags:** `-XX:+FlagName` enables, `-XX:-FlagName` disables
+ **Parameter flags:** `-XX:FlagName=something`

Ergonomics: Automatically tuning flags based on the environment.

## Notes on Premature Optimization

Premature optimization means that you don't complicate the code structure in order to obtain a small performance benefit from it. However, it does NOT mean that you write code in a way that is not performant, by picking "simpler" or less performant (but perhaps slightly more easy to understand or reason about) data structures or unstructured code which performs badly.

> Every line of code involves a choice, and if there is a choice between two simple, straightforward ways of programming, choose the more performant one.

Example code:

```java
logger.log(Level.INFO, "Value of x is: " + computeX() " and y is: " + computeY());
```

The code performs an unnecessary string concatenation which resolves both computations, which don't really result in anything that actually gets logged if the log level in production is less verbose than `INFO`.

Improved code:

```kotlin
if (log.isLoggable(Level.INFO)) {
	val x = computeX()
	val y = computeY()
	logger.log(Level.INFO, "Value of x is $x and y is $y")
}
```

## Finding Bottlenecks

The application server (or JVM tuning) may not be the bottleneck or source of performance issues. In a large enough system, the problem could be in a whole bunch of places, and good places to look at the database / network calls, external systems and anything that was newly added to the infrastructure.

## Optimize for the common case

+ Profile it and focus on the operations that take the most time (this does not mean you only look at leaf methods).
+ Occam's Razor: the simplest possible explanation is most likely the correct one. It's much more likely that it's a new configuration / code change that's causing performance issues, as opposed to a bug in the JVM/OS.
> When you hear hoofbeats, think horses, not zebras. *~ Medical diagnosis adage.*
+ Write simple, performant algorithms for the most common operations in your application. Optimize code paths that most users will take (even if it sometimes means that other less common code paths might get negatively impacted).

----

## References
1. [Java Performance: The Definitive Guide](https://www.oreilly.com/library/view/java-performance-the/9781449363512/) Chapter 1