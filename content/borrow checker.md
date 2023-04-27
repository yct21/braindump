---
title: "borrow checker"
tags:
- evergreen
- rust
---

[[rust|Rust]] implements  an _affine type system_ with _borrow checker_ to keep track of a [[variable|variable's]] state, and prevent invalid changes of the state.

Compiler could trace a value that owned by a variable by tracking the variable, which forms the **flow** of this value. Whenever the variable is accessed, borrow checker analyze if the access is valid in the flow. 

A value has these states:

- uninitialized
- normal
- value shared borrowed
- value exclusively borrowed
- value moved (not applicable to type that implements `Copy`)

A value could enter state `shared borrowed` or `exclusively borrowed` when it get referenced. Other status changes most happen during an assignment.

## Status tracking during an assignment

The borrow checker enforces different rules of assignment depending on whether the type of the variable implements `Copy` trait or not.

### Types implements `Copy` trait

When the type of a variable implements `Copy` trait, the variable is at one or several of:

- uninitialized
- normal
- value shared borrowed
- value exclusively borrowed

> [!question] How could a place be at several states at the same moment?
> 
> A place may enters multiple states due to branches or loops.
> ```rust
> let mut x = Box::new(42);
> let r = &x; // x is shared borrowed
> if rand() > 0.5 {
>     *x = 4;  // x ends the lease here
> } else {
>     println!("{}", r); // access of reference
> }
> 
> // now x is at both states of normal and shared borrowed
> ```

During an assignment of `a = b` with type `T` that implements `Copy`, compiler would check:

- `a` has to be mutable or at state of `uninitialized`.
```rust
let a = 1;
let b = 2;
a = b; // error: cannot assign twice to a immutable value
```
- If `a` is at state of `value shared borrowed` or `value exclusively borrowed`, `a` should be able to end the lease immediately.
```rust
let mut a = 1;
let ref_a = &a;
let b = 2;
a = b; // error: cannot assign to `a` because it is borrowed
println!("{}", ref_a); 
```
- `b` should not be at state of `uninitialized`.
- If `b` is at state of `value exclusively borrowed`, `b` should be able to end the lease immediately.
```rust
let mut a = 1;
let mut b = 42;
let ref_b = &mut b;
a = b; // error: cannot use `b` used it is mutably borrowed
println!("{}", ref_b);
```

After the assignment, bit-wisely copy the value from `b` to `a`, and `a` enters state `normal`.

### Types not implement `Copy` trait

When the type does not implement `Copy` trait, the variable is at one or several states of:

- uninitialized
- normal
- value shared borrowed
- value exclusively borrowed
- value moved

During an assignment of `a = b` with type `T` that does **not** implements `Copy`, compiler would check **all** the rules for [[borrow checker#What if it implements `Copy`?|Copy types]]. Additionally:

- `b` should not at state of `value moved`
- When `b` is at state of `value shared borrowed`, `b` should able to end the lease immediately.

After the assignment:

1. Drop the value owned by `a` if it does. 
2. Bit-wisely copy the value from `b` to `a`
3. `a` enters state `normal`.
4. `b` enters state `value moved`.

