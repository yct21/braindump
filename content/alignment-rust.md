---
title: "alignment (rust)"
tags:
- evergreen
- rust
---

All values in [[rust|Rust]] have an _alignment_  and size.

The alignment specifies what addresses are valid to store the value at. It is measured in bytes, and always a power of 2.

All values must be at least byte aligned. Some value have stringent alignment due to CPU and memory system that access memory in blocks larger than a single block.

---

> [!question]- Why should we care about alignment? 
> 
> Operations on data that is not aligned are referred to as misaligned accesses and can lead to poor performance and bad concurrency problems. For this reason, many CPU operations require, or strongly prefer, that their arguments are naturally aligned. A naturally aligned value is one whose alignment matches its size.

> [!question]- How could I check a value's alignment?
> 
> It can be checked with `std::mem::align_of_val` function.