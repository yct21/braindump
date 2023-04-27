---
title: "dynamic sized type"
aliases:
- DST
tags:
- evergreen
- rust
---

Some types in [[Rust]] does not have a size known at compile time. They are called _dynamic sized types_, and cannot implement `Sized` trait.

The most common dynamic sized types are:
- [[trait object]]
- [[slice]]