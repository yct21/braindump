---
title: "pin"
tags:
- evergreen
- rust
- asyncronous-programming
---

> [!todo] Todo:
> - [ ] no-alias

---

`Pin` is a smart pointer. It wraps another pointer and declares that its pointee has a stable location in memory, and cannot be moved elsewhere or deallocated before `Pin` is dropped, unless it wraps a types that implements `Unpin`.

Actually `Unpin` is an auto trait. So that most types are not limited by `Pin` even it is pinned. Therefore it is used in only a few situations.

## Why do we need `Pin`?

`Pin` was introduced to support self-referential structure for [[asynchronous programming-rust|asynchronous programming]].

The code below[^Self-referential types for fun and profit] contains a self-referential struct:

```rust
struct ParseState { 
    data: [u8; 4096], // the buffer we're parsing 
    current_token: &'data [u8], // current token during processing 
}
```

Since `current_token` is a slice from `data` field, we cannot assign a valid [[lifetime]] for `current_token` unless using `static`, which is not useful. Besides, when a value of `ParseState` is moved, `current_token` points to an invalid memory. 

In pre-async days of Rust, there are some techniques to avoid self-referential structure, eg. guard field, or using index instead of pointer type. However all these techniques come with costs, like heap allocation or locks. Since rustc heavily relies on self-referential  when desugaring [[async-await]] syntax, those costs are not acceptable. 

```rust
let fut = async {
    let mut x = xxx;
    let p = &mut x;
    // `x` and `p` are saved in the same enum variant field 
    // enum Generator { DoSth1(x, p); End };
    do_something1().await;
    // when resumed, `x` and `p` should be still valid
    do_something2(p); 
};

```

Therefore, we need `Pin` to ensure fields in self-referential structure to be valid whenever they are accessed before `Pin` is dropped.

## How could `Pin` ensure its pointee to be unmovable?

It actually can't. `Pin` declares a contract. It does not have any mechanism to prevent its pointee to be moved.

`Pin` is just a plain wrapper, without any compiler's black magic:

```rust
pub struct Pin<P> {
    pointer: P,
}
```

The "unmovable" contract should be guaranteed by the user of `Pin`, as described in document of `Pin::new_unchecked`:

> [Safety](https://doc.rust-lang.org/stable/std/pin/struct.Pin.html#safety)  
> This constructor is unsafe because we cannot guarantee that the data pointed to by `pointer` is pinned, meaning that the data will not be moved or its storage invalidated until it gets dropped. If the constructed `Pin<P>` does not guarantee that the data `P` points to is pinned, that is a violation of the API contract and may lead to undefined behavior in later (safe) operations.


## What is `Unpin`?

`Unpin` is an auto trait that is opt-in to implement. Any type `T` that implements `Unpin` trait can be safely moved after pinned with `Pin<Pointer<T>>`.

`Pin<&mut T>` only provides `get_mut` for `T: Unpin`. It makes unique reference of  types that does not implements cannot be fetched through `Pin` API.

> [!info] `noalias` and `Unpin`
> 
> Although `Pin` does not apply some black magic, the `Unpin` does. The compiler treats `Unpin` specially that for `T: !Unpin`, `&mut T` does not have to be `no-alias`.[^pin-talk]

[^pin-talk]: [不知道做点啥，今天来聊聊Pin吧](https://zhuanlan.zhihu.com/p/534089920)
[^Self-referential types for fun and profit]: [Self-referential types for fun and profit](https://morestina.net/blog/1868/self-referential-types-for-fun-and-profit)


