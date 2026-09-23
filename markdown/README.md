# Tesla RT (`rt`) Documentation Set

FreeRTOS-style documentation for **Tesla RT (`rt`)**, the open-source realtime kernel published as RTNG/`rt` (crate [`rt`](https://crates.io/crates/rt) on crates.io, Apache-2.0, author Chris Copeland).

> **Identity note.** This is the open-source kernel at [https://git.rtng.org/rt/rt](https://git.rtng.org/rt/rt). It is often referred to as Tesla’s realtime OS in partner enablement contexts. It is **not** a closed commercial SKU, and these docs make **no** claims about Tesla vehicle production software.

Source of truth for this doc set: local clone `/workspace/rtng-rt` (version **0.20.1** per `Cargo.toml`).

## How to browse

Start at the overview, then follow the numbered sections:

| # | Page | Contents |
|---|------|----------|
| 00 | [Overview](00-Overview.md) | Landing page and full TOC |
| 01 | [Introduction](01-Introduction.md) | What `rt` is and design goals |
| 02 | [Getting Started](02-Getting-Started.md) | Clone, SCons host build, simple example |
| 03 | [Kernel Features](03-Kernel-Features/) | Tasks, scheduling, sync primitives |
| 04 | [Interrupts](04-Interrupts.md) | ISR rules and nonblocking APIs |
| 05 | [Memory and Static Allocation](05-Memory-and-Static-Allocation.md) | Stacks, static objects, MPU |
| 06 | [Architecture and Ports](06-Architecture-and-Ports.md) | arm, riscv, aarch64, c28, signal |
| 07 | [Building and Examples](07-Building-and-Examples.md) | SConstruct, examples, cycle benches |
| 08 | [C / C++ / Rust](08-C-CXX-Rust.md) | Language bindings overview |
| 09 | [API Reference](09-API-Reference/) | Per-header summaries |
| 10 | [Porting Guide](10-Porting-Guide.md) | High-level porting patterns |
| 11 | [FreeRTOS / Zephyr Comparison](11-FreeRTOS-Zephyr-Comparison.md) | Talk-oriented comparison table |

Official upstream Rust docs (bindgen-generated): [https://docs.rs/rt](https://docs.rs/rt).

## Conventions used here

- API names are quoted exactly as in `include/rt/*.h` (`rt_mutex_lock`, `RT_TASK`, …).
- Where behavior is not documented in-tree, pages say **not documented in-tree**.
- Priority convention: **0 = highest**, **31 = idle** (`RT_TASK_PRIORITY_IDLE`).
