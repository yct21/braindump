---
title: "leaf future"
tags:
- evergreen
- rust
- asyncronous-programming
---

A leaf future is a [[future]]  without any inner futures. It directly represents some resource that may not yet be ready to return a result. 

```rust
// stream is a leaf-future 
let mut stream = tokio::net::TcpStream::connect("127.0.0.1:3000");
```

Typically leaf futures are provided by an asynchronous runtime as API. User of the runtime could combine these leaf futures with [[async-await]] syntax to construct complex tasks.