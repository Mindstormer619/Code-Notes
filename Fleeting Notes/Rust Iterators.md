# Rust Iterators
> Created: 2023-02-03 17:57

```rust
fn main() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    for val in v1_iter {
        println!("Got: {}", val);
    }
}
```

The `Iterator` trait only requires implementors to define one method: the `next` method, which returns one item of the iterator at a time wrapped in `Some` and, when iteration is over, returns `None`.

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
}
```

Calling `next()` changes the internal state of the iterator -- which is why `v1_iter` needs to be `mut`. In the `for` loop example, the loop takes _ownership_ of the iterator, which is why it does not need to be `mut`.

The `iter` method produces an iterator over immutable references. If we want to create an iterator that takes ownership of `v1` and returns owned values, we can call `into_iter` instead of `iter`. Similarly, if we want to iterate over mutable references, we can call `iter_mut` instead of `iter`.

**Consuming adaptors.** Methods that call `next` are called _consuming adaptors_, because calling them uses up the iterator. One example is the `sum` method, which takes ownership of the iterator and iterates through the items by repeatedly calling `next`, thus consuming the iterator. As it iterates through, it adds each item to a running total and returns the total when iteration is complete.

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();
        // can't use v1_iter after this point

        assert_eq!(total, 6);
    }
}
```

**Iterator adaptors.** _Iterator adaptors_ are methods defined on the `Iterator` trait that don’t consume the iterator. Instead, they produce different iterators by changing some aspect of the original iterator. e.g. The `map` method returns a new iterator that produces the modified items.

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];

    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

    assert_eq!(v2, vec![2, 3, 4]);
}
```

Iterators are **zero cost abstractions** in Rust! Use them.

----

## References
1. https://doc.rust-lang.org/book/ch13-02-iterators.html