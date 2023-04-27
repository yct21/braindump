---
title: "wide pointer"
tags:
- evergreen
- rust
---

A wide pointer is a pointer that contains an extra word-sized field that gives the additional information about the value it points. 

---

> [!question] Why we need wide pointer?
> 
> Rust requires types to be `Sized` nearly everywhere. This restriction is so common that `T: Sized` is bounded by default unless explicitly opt out with `T: ?Sized`.
When dealing with a [[dynamic sized type]], a wide pointer is required for compiler to generate reasonable code for manipulate the value of DST.

> [!tip]  `Box` and `Arc` also support storing [[dynamic sized type]].