# Rust Copy Types
> Created: 2023-01-03 22:11
> #rust

Copy types are basically the primitives and anything that is composed of other Copy types.

Anything that implements the `Copy` trait is instantiated on the stack, and can therefore be trivially copied. In other words, the following code works:

```rust
let x = 4;
let y = x;

println!("{x} and {y}");
```

This would not work if the type of `x` (which is `i32` in this case) did not implement `Copy`, as `x`'s value gets _moved_ to `y` and therefore goes out of scope, and cannot be referenced in the `println!()`.

----

## References
1. https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy