2024-04-26 09:43

Status: #final 

Tags: [[Rust]]

# Ownership
Ownership is a set of rules enforced by the compiler to guarantee that a program is memory-safe. If any of these rules is violated the program won't compile.

Before diving into Ownership, lets revisit some primary concepts.

### The stack and the heap
Both stack and the heap are memory locations available to a program during execution. The stack stores values in order, through a *last in, first out* system, and only accept fixed-sized variables. When the size of a variable is unknown at compile time, the program uses the heap to store those values.

The stack is much faster memory to access than the heap, since all items are store in a sequential form, to access the next value of the stack, the program will simply pop it off. The access and allocation to heap data is more time-costly, since the allocator needs to find an empty space in the heap to allocate the exact amount of data being requested, and later on, to access the data, it needs to follow a pointer to retrieve the value stored. 

> [!NOTE]
>Ownership only deals with dynamic-sized variables. That is, variables on the heap

Keeping track of parts of the code that are using data on the heap, the amount of duplicated data on the heap, and cleaning up unused data on the heap are all problems that ownership addresses.

### Ownership rules
1. Each value has an owner
2. A value can only have one owner at a time
3. When the owner goes out of scope, the value will be dropped

### Memory and allocation
`String` is a dynamic string type, which means that contrary to the regular string type, that is a fixed-size defined at compile time, the value of String can be unknown ahead of time, and only be created during the run-time.

If we think in terms of ownership of a heap-allocated variable, a new ownership is created at the moment that a variable is declared and it lasts until the end of current scope.

```rust
{
	let s = String::from("hello"); // s is valid from this point forward
	
	// do stuff with s
}

// the scope is now over, and s is no longer valid
```

In the example above, the ownership is created when the `String::from("hello")` is assigned to variable `s`, being `s` the owner of that piece of data on the heap.

When the value goes out of scope, a special function called `drop()` is called automatically by Rust, in which the memory is returned to the heap, and it's freed to be used by another portion of the code.

Consider the two snippets below:

```rust
let x = 5;
let y = x;
```
```rust
let x = String::from("hello");
let y = x;
```

At first sight, they seem to be doing the same thing, assigning a value to `x` and binding the value of `x` to `y`. That's what's happening only on the snippet one, where the size of the variable `x` is known at compilation time, therefore this variable is stored in the stack and can be easily copied by Rust to another variable, in this case `y`.

In the second snippet, the value of the string is being allocated to the heap which returns a pointer to the variable and when the value of `x` is assigned to `y`, we end up with two pointers on the stack. The problem here lies on the call to `drop()` function, that would try to deallocate the same heap location twice, which would lead to a double-free bug and can cause memory corruption as well as open the door for program exploitation.

To avoid that, when a pointer is used the instantiate another Rust understands it as a "move", where the ownership moves from one variable to another and the previous owner gets invalidate.

## References

