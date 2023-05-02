---
title: "interior mutability"
tags:
- evergreen
- rust
---

In Rust some types provide _interior mutability_, that allows mutating a value through a [[reference|shared reference]].

These types normally fall into two categories:

- `Mutex`, `RefCell`: These types contain safety mechanisms to ensure that the value it contains is lent out at most once (mutably) at any given moment.
- `atomic`, `Cell`: These types does not lend out the inner value but provide methods to manipulate value in place.