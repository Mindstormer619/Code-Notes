# Simplicity
> Created: 2023-02-13 19:49
> This is **Simplicity** in the sense of maintainable software systems.

**Simplicity** refers to designing systems in ways that make it easy for new engineers to understand the system's operations. Making a system simpler does not mean reducing functionality. We generally try to remove *accidental complexity*. Complexity is accidental if it is not inherent in the problem the software solves, but arises only from the implementation.

One of the best tools for creating simplicity is **abstraction**. A good abstraction hides a lot of implementation detail behind a clean facade. This helps in code reuse and understanding, and higher quality software. However, beware over-abstracting or using the wrong abstractions. Finding good abstractions can be very hard.

> *Duplication is far cheaper than the wrong abstraction.*
> ~ [Sandi Metz](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)

> 💡 **Big ball of mud.** A software system mired in complexity and interdependent systems which are hard to understand or unravel.



----

## References
1. [[Designing Data Intensive Applications - Reliable, Scalable, Maintainable Apps#Maintainability]]