---
title: "async/await"
tags:
- evergreen
- rust
- asyncronous-programming
---

> [!todo] todo:
> 
> - [ ] relationship with future trait

---

The `async/await` are syntax sugars for [[asynchronous programming-rust|asynchronous programming]] in Rust. 

- An `async fn` declares a function that return a value that implements a [`Future` trait](future.md). 
```rust
// `foo()` returns a type that implements `Future<Output = u8>`. 
// `foo().await` will result in a value of type `u8`.
async fn foo() -> u8 { 5 }
```
- An `async` block declares a type that implements a `Future` trait.
```rust
```rust
fn bar() -> impl Future<Output = u8> {
    // This `async` block results in a type that implements 
    // `Future<Output = u8>`.
    async {
        let x: u8 = foo().await;
        x + 5
    }
}
```

> [!question] Why do we need `async/await` syntax?
> 
> It preserves look and feel of ordinary synchronous programming.

## How does compiler generate code for async function or block?

A future constructed with async/await syntax is implemented as a generator with a state machine.

> [!info] Generator
>
> A generator is a routine that can be used to control the iteration behavior of a loop. 
> 
> - When running, it can yield control back to caller (and optionally return a value). 
> - When this generator is called again, it resumes its control flow from where it last yielded.
>

### From `await` to state machine

The `await` syntax separates a future into several continuations. Given code below:

```rust
async fn some_async_process<T>(rx: Receiver<T>, tx: Sender<T>) {
    let data1 = Step1::process().await;
    let data2 = Step2::process(&data1).await;
    let data3 = Step3::process(&data2).await;

    while let Some(t) = rx.next().await {    
        tx.send(Step4::process(&t, &data3).await).await;
    }
}
```

The compiler would generate a state machine like this:

![[images/async-await-statemachine.png]]

Each node in state machine can be treated like a variant of enum. It would also store all variables in the continuation as its internal data. This makes this state machine like a enumeration:

> [!tip] This code is inaccurate.
> 
> The code below is a demonstration of the mechanism. Rustc would perform several optimizations during a real process.

```rust
// Please replace variables below with its type
enum StateMachine {
    Begin(rx, tx),
    Step1(rx, tx),
    Step2(rx, tx, data1),
    Step3(rx, tx, data1, data2),
    RxNext(rx, tx, data1, data2, data3),
    Send(rx, tx, data1, data2, data3, t),
    End,
}
```

The enumeration is generated so that the future could match its variants.

### Match the enumeration

When running (polling) a future, it matches its variants first, then decides which continuation to run. 

It works like this:

```rust
// Actually it would compiles to Future::poll
fn some_async_process<T>(rx: Receiver<T>, tx: Sender<T>) {
    let state = StateMachine::Begin(rx, tx);
    while state != StateMachine::End {
        match state {
            // do computations and modify `state` in each continuation
            Begin => { ... },
            Step1 => { ... },
            Step2 => { ... },
            Step3 => { ... },
            RxNext => { ... },
            Send => { ... },
        }
    }
}
```

The code above describes how program decides where to resume. What happens when it yields is described in [[future|Future]].




