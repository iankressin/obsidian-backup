---
date: 2024-05-09 07:26
tags:
  - final
source: https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html
---
# Lifetimes
Lifetimes is a special type of generic in rust. Unlike the regular type generics, which expresses that a given value is open to assume any type provided by the caller, lifetime generics are about the lifespan of a value inside a given scope.

Every value inside a Rust program has a its own lifetime, as mentioned above, is the scope in which the reference exists and it's valid.

The borrow-checker is responsible for validating lifetimes, either they're implicit or explicitly defined. 

### Lifetime elision
In order to make Rust more ergonomic, the compiler allows the code to *elide* the declaration of lifetimes, when it's possible to infer it. The example below shows a elided and an expanded version of the same function.
```rust
fn substr(s: &str, until: usize) -> &str; // elided
fn substr<'a>(s: &'a str, util: usize) -> &'a str; // expanded
```
The elision above is made possible because we the lifetime being returned must have the same lifetime as the reference being passed as parameter, since that's the only reference accepted as param.

Suppressing the lifetimes from the function below, otherwise, it's not possible.
```rust
fn longer_str(s1: &str, s2: &str): &str; // illegal
fn longer_str<'a>(s1: &'a str, s2: &'a str): &'a str; // valid
```
The first signature doesn't make it clear for the borrow checker for how long the reference being returned will be valid for, and the borrow checker is also not able to guess it, since it doesn't which possible lifetimes `s1` and `s2` can have.
To fix that, we need to annotate the lifetime of both references. In this case we're telling the compiler that both `s1` and `s2` have a common lifespan, and the lifetime of the returned value is the same as the shortest of both.

### Lifetimes on structs
Rust also allows to assign references to a field of a struct, with the caveat that we need to annotate the lifetime of each reference being used.
```rust
struct Car<'a> {
	door: &'a Door
}
```
The reference above tells the compiler the moment that the code spins up a new `Car`, the to `Door` must live past `Car`.

We can also have multiple lifetimes in a struct. Now car have both a `Wheel` and a `Door`, and both must live longer than `Car`.
```rust
struct Car<'a, 'b> {
	door: &'a Door,
	wheel: &'b Wheel,
}
```
### Static lifetimes
A static lifetime is given to references that lives throughout the whole execution of the program, and because of that they are built into to the binary. When a program starts, a piece of memory  is allocated these values, and the same pointer is used to reference the value through out the whole execution.

In the example below, we have a `struct` that holds a string literal, and for that reason, `api_key` receives the `'static` lifetime.

The function `get_config` must annotate its return type as `'static` since it returns a borrowed value.
```rust
struct Config {
    api_key: &'static str,
    timeout: u32,
}

static CONFIG: Config = Config {
    api_key: "secret",
    timeout: 30,
};

fn get_config() -> &'static Config {
    &CONFIG
}
```
### Lifetime bounds
Is possible to explicitly tell the compiler that a reference outlives another.
```rust
fn func<'a, 'b>(x: &'a str, y: &'b str) where 'a: 'b;
```
## References

https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html
https://www.youtube.com/watch?v=rAl-9HwD858