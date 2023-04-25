---
title: "asynchronous programming (rust)"
tags:
- evergreen
- rust
- asyncronous-programming
---

> [!todo] Todo:
> - [ ] compare with other concurrent programming model

---

_Asynchronous programming_ is a concurrent programming model to run multiple tasks concurrently with non-blocking interfaces.

In [[rust|Rust]] asynchronous programming is implemented through [[future|Future]] and [[async-await|async/await syntax]] to  preserving look and feel of ordinary synchronous programming.

Specifically:

- Rustc would [[async-await#How does compiler generate code for async function or block?|generate code]] for `async` function or closure.
- Rust standard library provides interfaces like `Future` or `Waker`.
- The asynchronous runtimes are provided by 3rd party library like [tokio](https://tokio.rs) or [async-std](https://github.com/async-rs/async-std). 

Most 3rd party asynchronous runtime are implemented with reactor-executor pattern that run tasks as stackless coroutines, since `async/await` syntax and `Future` trait are tailored into this approach.

> [!question] Why Rust community prefers stackless coroutine
> 
> Rust community adopted stackless coroutine for these reasons:
> 1.  It's easy to convert normal Rust code to a stackless coroutine using [[async-await|async/await]] as keywords (it can even be done using a macro).
> 2.  No need for context switching and saving/restoring CPU state.
> 3.  No need to handle dynamic stack allocation.
> 4.  Very memory efficient.
> 5.  Allows us to borrow across suspension points.