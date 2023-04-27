---
title: "unsafe (rust)"
tags:
- evergreen
- rust
---

_Unsafe_ is the mechanism that Rust provides to take advantage of the invariants that, for whatever reason, the compiler cannot check.

> Crucially, unsafe code is not a way to skirt the various rules of Rust, like borrow checking, but rather a way to enforce those rules using reasoning that is beyond the compiler. When you write unsafe code, the onus is on you to ensure that the resulting code is safe[^rust for rustaceans].

[^rust for rustaceans]: rust for rustaceans