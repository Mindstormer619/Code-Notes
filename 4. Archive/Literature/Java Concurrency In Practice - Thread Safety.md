# Java Concurrency In Practice - Thread Safety
> Created: 2023-05-30 19:05
> #book 

Writing thread-safe code is, at its core, about managing access to *shared mutable state*.

Whether an objects needs to be thread-safe depends on how it will be _used_; it is not a property of what the object _does_, rather whether it will be accessed from multiple threads. Making the object thread-safe requires using synchronization to coordinate access to mutable state.

**Whenever more than one thread accesses a given state variable, and one of them might write to it, they must ALL coordinate their accesses to it using synchronization.**

> ❗ If multiple threads access the same mutable state variable without appropriate synchronization, **your program is broken**. There are three ways to fix it:
> 
> + **Don’t share** the state variable across threads;
> + Make the state variable **immutable**; or
> + Use **synchronization** whenever accessing the state variable.

When designing thread-safe classes, good OO techniques -- **encapsulation**, **immutability**, and **clear specification of invariants** -- are your best friends. The less code that has access to a given variable, the easier it is to ensure proper synchronization.



----

## References
1. Book: Java Concurrency In Practice Chapter 2