---
title: "lifetime"
tags:
- evergreen
- rust
---

A _lifetime_ is a name for a region of code that some reference must be valid for.

[[borrow checker|Borrow checker]] would trace the path of a reference from where it borrows value from the owner, to the point of each access, and make sure there are no conflicting uses along that path.

> [!info] Lifetime does not have to be contiguous.
> 
> ```rust
> let mut x = Box::new(42);
> let r = &x; // 'a
> if rand() > 0.5 {
>     *x = 4;  // not 'a
> } else {
>     println!("{}", r); // 'a
> }
> ```
> 
> ```rust
> let mut x = Box::new(42);
> let mut z = &x;    // 'a
> for i in 0..100 {
>     println!("{}", z);  // 'a 
>     x = Box::new(i);    // not 'a
>     z = &x;             // 'a
> }
> 
> println!("{}", z);      // 'a
> ```