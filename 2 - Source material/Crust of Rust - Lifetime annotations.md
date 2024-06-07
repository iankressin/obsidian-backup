2024-04-30 06:54

Link: https://www.youtube.com/watch?v=rAl-9HwD858

# Crust of Rust - Lifetime annotations
- we can always assign a lifetime that's equal or longer than another lifetime
```rust
// assume x has lifetime 'a and type &str
let x = ""; // this is valid since "" has static lifetime, that's the longer lifetime of them all
```
- static lifetimes lives on the program binary, so when the OS loads the program, it allocates a piece of memory for that constant value, and when required it always returns the same pointer to that memory location
- we can specify that one lifetime is greater than the other by using:
```rust
where 'a: 'b // This is saying that a lives past b
```
Resume in: https://youtu.be/rAl-9HwD858?si=Jb_eEgvQNT_3o5FK&t=4764