# Rust Lifetimes
> Created: 2023-01-06 21:39
> #rust

In Rust, we annotate _lifetimes_ with `'a` for a lifetime parameter named `a`.

## What are Lifetimes?

A lifetime indicates how long a reference is expected to stay alive. The lifetime of a reference cannot cross the scope boundary of the thing it is referencing — if this is at all possible, Rust will prevent the code from compiling.

## Lifetime parameters

```rust
fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str  
where
   T: Display,
{  
   println!("Announcement! {}", ann);  
   if x.len() > y.len() {  
      x  
   } else {  
      y  
   }  
}
```

In the above example, both `x` and `y` have the lifetime `'a`, which matches the lifetime of the output parameter. That means, the output parameter is expected to live as long as the shorter lifetime of the two references `x` and `y`.



----

## References
1. https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html