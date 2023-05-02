---
title: "future"
tags:
- evergreen
- rust
- asyncronous-programming
---

> [!todo] Todo:
> 

---

A future represents an [[asynchronous programming|asynchronous]] computation that can produce a value at some time. 

In rust a future is a type or value of type that implements `std::future::Future` trait.

> [!question] Why do we need a `Future` trait in standard library?
> 
> The `Future` trait are building blocks of asynchronous programming in Rust. They provides a standardized interface for user and library authors. Without it, all libraries with an asynchronous interface may come with their own `polling` method, all with different names, signatures and return types.

## How do we use future?

Most asynchronous runtime would come up with their own [[leaf-future|leaf future]] as API. And many libraries have asynchronous interfaces. We could use [[async-await]] to compose the these APIs into a task and run it on the runtime.

```rust
use mini_redis::{client, Result};
use tokio::fs::File;
use tokio::io::{self, AsyncReadExt};

#[tokio::main]
async fn main() -> Result<()> {
    // Leaf futures from tokio
    let mut f = File::open("data.txt").await?;
    let mut buffer = [0; 10];
    let n = f.read(&mut buffer[..]).await?;

    // Asynchronous APIs from mini-redis
    let mut client = client::connect("127.0.0.1:6379").await?;
    client.set("hello", "world".into()).await?;
    let result = client.get("hello").await?;
    println!("got value from the server; result={:?}", result);

    Ok(())
}

```

## How does a future run?

### Polling

Futures in Rust use a `poll` based approach. The asynchronous interface of Rust is a method that returns a `Poll` type, which is actually the result of polling:

```rust
// here your result or come back later
enum Poll<T> {
    Ready(T),
    Pending
}
```

The `Future` trait is an abstraction over this pattern:

```rust
pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Calling `poll` method of a future would drive the future as far towards completion as possible. When it returns a `Poll` struct as result, it tells the caller that the result is ready or not. When it is not ready, the caller could do something else in the meantime rather than having to go to sleep until some particular situation changes.

A typical asynchronous runtime would poll the future at some appropriate time during managing the lifecycle of this future.

### A future's lifecycle

A typical asynchronous runtime (tokio or async-std) that implements reactor-executor pattern would manage a future with theses phases:

1. Poll phase: The future is polled by an [[asynchronous executor|executor]], until a point that can no longer make progress. When it returns `Poll::Ready`, the future is completed with this outcome, otherwise it goes to wait phase.
2. Wait phase: The future is registered in an event queue by [[asynchronous reactor|reactor]] and goes inactive, until some event happens that push the future to wake phase.
3. Wake phase: Some event happens. The waker calls `waker()` to indicate the executor that the future is ready to be polled again. Then the executor would update its internal queue to hold this future.  When the future is scheduled, it goes to poll phase.

The runtime drives a future into its next phase of lifecycle until it completes. That's how a future runs.

> [!question] A waker is supposed to call `wake` on future when some event happens. How does this work exactly?
> 
> It depends on the future.
> 
> When a future returns `Poll::pending` from polling and enters wait phase, it is the future's responsibility to ensure that something calls `wake` on the provided `Waker` when the future is able to make progress.
>
> The most common method are spawning another thread or using `Epoll`, `Kqueue` and `IOCP`.

> [!question] What is this `Context` in `poll`'s arguments?
> 
> The `Context`  is a type that only wraps a [[asynchronous waker|waker]], but it gives flexibility for future evolution of API in rust.

> [!danger] Don't poll a future after it has returned `Poll::Ready`
> 
> It is forbidden to poll a finished future. Going against it may cause panic or undefined behaviors.