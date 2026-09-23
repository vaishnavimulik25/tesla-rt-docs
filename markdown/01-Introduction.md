# Introduction

## Identity

**Tesla RT (`rt`)** is the open-source realtime kernel known upstream as **RTNG/`rt`** (crate `rt` on crates.io). Partner materials sometimes call it Tesla’s realtime OS. This documentation set describes **only** the public Apache-2.0 tree; it does not assert anything about Tesla vehicle production firmware.

| | |
|--|--|
| Repository | https://git.rtng.org/rt/rt |
| Version documented | **0.20.1** |
| License | Apache-2.0 |
| Author | Chris Copeland |
| Language surface | C (primary), C++ wrappers (`cxx/`), Rust crate (`rust/`) |

## Design goals (from `README.md` and headers)

1. **Full preemption** — higher-priority ready tasks preempt lower-priority ones; yield among peers at the same priority.
2. **Atomics-based synchronization** — mutexes, semaphores, events, queues, etc. lean on C11 atomics (`rt/atomic.h`) for high performance and preemptibility.
3. **ISR-safe nonblocking paths** — non-blocking interfaces are usable from both tasks and interrupts. Semaphores and events provide explicit `*_from_task` / `*_from_interrupt` variants when the call site is known.
4. **Avoid IRQ-disable for sync where possible** — the README states `rt` does not rely on disabling interrupts to implement synchronization. On architectures without hardware atomics, interrupt masking is used to emulate atomics with **very short** gaps in interrupt availability.
5. **Static allocation first** — tasks, stacks, and sync objects are typically declared with macros (`RT_TASK`, `RT_MUTEX`, `RT_SEM`, …) that place storage at compile/link time.

## What `rt` is (and is not)

**Is:**

- A small, preemptible RTOS kernel with a rich sync set (mutex + priority donation, recursive mutex, semaphore, condvar, event bits, MPMC lock-free queue, notifications, software timers, rwlock, barrier, once, pool, pend_function, …).
- Portable across Arm Cortex-M/R/A (32-bit), AArch64, RISC-V, TI C28x, plus a POSIX host simulator (`arch/signal`).
- Usable from C, C++, and Rust (bindgen + idiomatic wrappers).

**Is not (based on this tree):**

- A multi-hart / SMP scheduler — scheduling state is organized around a single `rt_active_task` and one ready-bit bitmap (`src/rt.c`). Treat stock `rt` as **largely single-hart**.
- A full OS with filesystem, networking, or process isolation beyond optional MPU task regions.
- A closed commercial product SKU.

## Core runtime model

```
constructors (RT_TASK / RT_* objects)
        │
        ▼
   rt_start()  ──►  rt_start_sched()  ──► first ready task
        │
        ▼
  tasks + interrupts
        │
   synchronous syscalls (SVC/ecall/…)
   pendable syscalls (PendSV / soft IRQ / signal)
        │
   tick via rt_tick_advance()
```

Tasks are created before `rt_start` (typically via `RT_TASK` constructor macros). Blocking operations enter the syscall path; interrupt-side posts often enqueue a **pendable** syscall record and trigger deferred processing so the kernel can wake waiters without long critical sections.

## Next

- [Getting Started](02-Getting-Started.md) — build and run on the host  
- [Tasks](03-Kernel-Features/Tasks.md) — task model and priorities  
