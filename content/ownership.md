---
title: "ownership"
tags:
- evergreen
- rust
---

Rust's [[memory|memory model]] centers on the idea that all values have a single owner. Exactly one place] is responsible for ultimately deallocating each value.

This model is enforced through [[borrow checker]].

> [!question] Why Rust enforce this memory model?
> 
> It allows Rust to be memory safe and efficient, while avoiding garbage collection. 