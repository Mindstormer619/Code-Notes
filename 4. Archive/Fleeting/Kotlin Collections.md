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

+ `HashMap` is implemented as an array of linked lists. Each element in the array is called a bucket and contains a linked list of entries.
+ A hash code is calculated for each key using the `hashCode()` method. The hash code is used to determine in which bucket the key-value pair should go.
+ Collisions are handled using chaining. This means if two keys hash to the same bucket, they are stored as a linked list in that bucket.
+ The load factor determines when to resize the hash map. The default load factor is 0.75, meaning once 75% of buckets have at least one entry, the hash map will resize.
+ Resizing works by creating a new bucket array of size `newCapacity` and rehashing all elements into this new array. This is an *O(n)* operation.
+ HashMap allows null keys and values. However, there can only be one null key in the map.
+ The `get()`, `put()`, `remove()`, `containsKey()` and `containsValue()` operations all run in *O(1)* time on average since we can hash directly to the bucket. However, resizing the map or iterating it requires *O(n)* time.
+ The initial capacity of a hash map is 16, and it's always a power of 2. Capacity is the number of buckets, not actual elements that can be stored.
+ Load factors over 0.75-0.8 usually don't improve space efficiency but only increase lookup times. So the default load factor of 0.75 provides a good space/time tradeoff.
+ `HashMap` is not thread-safe. `ConcurrentHashMap` is a thread-safe alternative.

### Differences with `Hashtable`

+ `Hashtable` is thread-safe via synchronization
+ `HashMap` allows adding one `Entry` with `null` as key, as well as `null` values -- `Hashtable` does not allow `null` at all.
+ Since JDK 1.8, `Hashtable` has been deprecated. Use `ConcurrentHashMap` -- does not synchronize over the whole map, for higher performance.

