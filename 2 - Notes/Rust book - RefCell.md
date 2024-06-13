---
date: 2024-06-13 09:04
tags:
  - "#inprogess"
source: https://doc.rust-lang.org/stable/book/ch15-05-interior-mutability.html
---


# Rust book - RefCell

- *Interior Mutability*: mutate a variable when there are immutable references to that data inside an unsafe block, and wrap it in a safe API and the outer type is still immutable

- RefCell enforces the borrow checker's rules at runtime, whilst Box enforces at compile time

- Read about "Halting Problem"

- Designed to be used in single threaded scenarios

- The interior mutability pattern can be used during tests, when a struct takes in a immutable struct/reference, but we want to mutate the reference internally in order to be able to test the fynct 
