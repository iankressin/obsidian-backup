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

- The interior mutability pattern can be used during tests, when a struct/functions takes in a immutable struct/reference. We can mutate the reference internally, but still pass as immutable reference.

```rust
trait MessageDispatcher {
    fn dispatch(&self, message: String);
}

struct MessageBuilder<'a, T: MessageDispatcher> {
    dispatcher: &'a T,
}

impl<'a, T> MessageBuilder<'a, T>
where
    T: MessageDispatcher,
{
    fn new(dispatcher: &'a T) -> Self {
        Self { dispatcher }
    }
}

impl<'a, T> MessageBuilder<'a, T>
where
    T: MessageDispatcher,
{
    fn build_and_send_message(&self, name: &str) {
        let message = format!("Hello, {name}");

        self.dispatcher.dispatch(message);
    }
}

#[cfg(test)]
mod test {
    use std::{
        borrow::{Borrow, BorrowMut},
        cell::RefCell,
    };

    use super::{MessageBuilder, MessageDispatcher};

    struct Messenger {
        message: RefCell<String>,
    }

    impl Messenger {
        fn new() -> Self {
            Self {
                message: RefCell::new(String::from("")),
            }
        }
    }

    impl MessageDispatcher for Messenger {
        fn dispatch(&self, m: String) {
            let mut message = self.message.borrow_mut();
            *message = m;
        }
    }

    #[test]
    fn test_build_message() {
        let messenger = Messenger::new();
        let builder = MessageBuilder::new(&messenger);
        builder.build_and_send_message("Ian");

        assert_eq!(messenger.message.into_inner(), "Hello, Ian");
    }
}
```
