# Kotlin Collections
> Created: 2023-06-10 12:06

Interfaces:

![[collections-diagram.png]]

## List: ArrayList

Resizable-array implementation of the `List` interface. **NOT synchronized!** 

Synchronization is typically accomplished by synchronizing on some object that naturally encapsulates the list. If no such object exists, the list should be "wrapped" using the [`Collections.synchronizedList`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedList-java.util.List-) method. This is best done at creation time, to prevent accidental unsynchronized access to the list:

```java
List list = Collections.synchronizedList(new ArrayList(...));
```

## Set: LinkedHashSet

The default implementation of `MutableSet` – [`LinkedHashSet`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-linked-hash-set/index.html) – preserves the order of elements insertion. Hence, the functions that rely on the order, such as `first()` or `last()`, return predictable results on such sets.

> An alternative implementation – [`HashSet`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-hash-set/index.html) – says nothing about the elements order, so calling such functions on it returns unpredictable results. However, `HashSet` requires less memory to store the same number of elements.

This implementation differs from `HashSet` in that it maintains a doubly-linked list running through all of its entries. This linked list defines the iteration ordering, which is the order in which elements were inserted into the set (_insertion-order_). Note that insertion order is _not_ affected if an element is _re-inserted_ into the set. (An element `e` is reinserted into a set `s` if `s.add(e)` is invoked when `s.contains(e)` would return true immediately prior to the invocation.)

This implementation spares its clients from the unspecified, generally chaotic ordering provided by [`HashSet`](https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html "class in java.util"), without incurring the increased cost associated with [`TreeSet`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html "class in java.util").

> 💡 **TreeSet.** A [`NavigableSet`](https://docs.oracle.com/javase/8/docs/api/java/util/NavigableSet.html "interface in java.util") implementation based on a [`TreeMap`](https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html "class in java.util"). The elements are ordered using their [natural ordering](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html "interface in java.lang"), or by a [`Comparator`](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html "interface in java.util") provided at set creation time, depending on which constructor is used. Note the difference with `LinkedHashSet`: it does NOT preserve insertion ordering, rather does natural ordering.
>
> This implementation provides guaranteed *log(n)* time cost for the basic operations (`add`, `remove` and `contains`). **Not Synchronized!**
> 
> TreeSet is implemented using a Red Black Tree: https://www.baeldung.com/java-tree-set

## Map: LinkedHashMap

The default implementation of `MutableMap` – [`LinkedHashMap`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-linked-hash-map/index.html) – preserves the order of elements insertion when iterating the map. In turn, an alternative implementation – [`HashMap`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-hash-map/index.html) – says nothing about the elements order.

`LinkedHashMap`: Hash table and linked list implementation of the `Map` interface, with predictable iteration order. This implementation differs from `HashMap` in that it maintains a doubly-linked list running through all of its entries. This linked list defines the iteration ordering, which is normally the order in which keys were inserted into the map (_insertion-order_).



----

## References
1. https://kotlinlang.org/docs/collections-overview.html#collection-types