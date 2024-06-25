---
date: 2024-04-26 10:12
tags:
  - final
source: https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html
---
# Rust book - Lifetimes

- Lifetimes validates a reference exists for as long we need them to
- Every value has a lifetime, which is the scope that the reference is valid
- The borrow checker is responsible for ensure the lifetime of each reference is valid in the moment its used
- Lifetimes annotations are meant to tell the compiler => borrow checker how they relate to other references. So a single generic lifetime annotation would make much sense in a function.
- The signature below tells the compiler that the return value will be valid as long as both parameters are:
```rust
fn longest<'a>(s1: &'a str, s2: &'a str) -> &'a str
```
- `valid = a >= b && b == c`
		where `a` is outer scope
			 `b` is inner scope
			 `c` is scope where function is defined

- Every time a function returns a reference it needs to specify the lifetime? No only when the compiler cannot figure out on its own
- Lifetime elision solves the problem of specifying lifetime annotations on functions that take a reference as param and return that same reference
- In the struct below, the struct can't live longer than the reference it holds
```rust
struct Book<'a> {
	page: &'a str,
}
```
- the `'static` lifetime tells the compiler that reference lives forever. That's the case of string literals, since they're put directly into the program binary. 