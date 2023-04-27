---
title: "memory (rust)"
tags:
- evergreen
- rust
---

In [[rust]] programming, a program may have access to a stack, a heap, registers, text segments, memory-mapped registers, memory-mapped files, and perhaps nonvolatile RAM. 

> [!info] Rust does not yet have a defined memory model.
> 
>  Various academics and industry professionals are working on various proposals, but for now, this is an under-defined place in the language[^rust reference: memory model].

## Memory regions

It's responsibility of standard library to adapt the operating system. In a specialized environment, heap or even stack may not exist at all.

There are some general rules:

-   There are no correspondences between a type to any memory region.
-   Static items are allocated in static memory.
-   [[variable|Variables]] in function are allocated in stack, except for temporaries being constant promoted, which are allocated in static memory.
-   There is a global allocator in Rust that is responsible to allocate and free memory by adapting memory management for a process of the operating system .
    -   Several types in standard library works tightly with global allocator (`Box` etc). They are considered to be API for programmers to access heap.
-   Some places are considered to be a fraction of a bigger place, so that they are not related to some specific memory region.
    -   fields in a `struct`
    -   variables in a `async` function
    -   variables captured by a closure

[^rust reference: memory model]: https://doc.rust-lang.org/reference/memory-model.html