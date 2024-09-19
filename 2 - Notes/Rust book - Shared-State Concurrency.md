---
date: 2024-09-04 07:51
tags:
  - "#inprogess"
source: https://doc.rust-lang.org/book/ch16-03-shared-state.html
---
# Shared-State Concurrency

#### Mutex
- *Mutual exclusion* (mutex) is a mechanism to manage multiple threads accessing the same memory region. Only one thread can access a piece of data at a time.
-  To access data in a mutex, a thread signals that wants to acquire the mutex lock. Lock is a data structure, part of the mutex, that keeps track of who's accessing the data at any given time. 
- Acquiring the lock must happen before using the data. 
- A thread must unlock the data after using, so other threads can access it. 
- This is an example of Mutex in action. 
```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {m:?}");
}
```
- After accessing the data, the thread gets a mutable reference to the value inside the lock.
- Type system ensures that a thread acquires the lock before using it, because `m` if of type `Mutex<i32>`
- In Rust, the Mutex type panics if someone else is holding the lock and a thread tries to get the hold of the lock. 

#### Atomic Reference Counting
- [[Rust book - Rc]] is not enough to share mutex references along threads, since it doesn't use any concurrency primitives, which can lead to make sure changes to counter can't be interrupted by another thread
	- Arc uses a concurrency primitive called atomic, which makes *Mutex* shareable across threads
```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```