# Tesla RT (`rt`) — Documentation Overview

Welcome to the Tesla RT (`rt`) documentation set, structured similarly to the [FreeRTOS Documentation Overview](https://www.freertos.org/Documentation/00-Overview).

## What is Tesla RT (`rt`)?

**Tesla RT (`rt`)** is the open-source RTNG/`rt` kernel — a fully preemptible realtime operating system built around atomics-based synchronization. Upstream: [https://git.rtng.org/rt/rt](https://git.rtng.org/rt/rt). Published as crate **`rt` 0.20.1** on crates.io (Apache-2.0, Chris Copeland).

Partner materials may call it Tesla’s RTOS. These docs describe the **open-source kernel and public BSP repos only**. They do **not** claim anything about closed vehicle production software.

From the project `README.md`:

> `rt` is a real-time operating system capable of full preemption. `rt` provides many familiar synchronization interfaces and implements them with atomics for high performance and preemptibility. `rt`'s non-blocking interfaces can be used safely in both tasks and interrupts, and it does not rely on disabling interrupts to implement synchronization.

## Table of contents

1. [Introduction](01-Introduction.md) — design goals and scope  
2. [Getting Started](02-Getting-Started.md) — host simulation with `arch/signal`  
3. **Kernel Features**
   - [Tasks](03-Kernel-Features/Tasks.md)
   - [Scheduling](03-Kernel-Features/Scheduling.md)
   - [Mutexes](03-Kernel-Features/Mutexes.md) (priority donation)
   - [Semaphores](03-Kernel-Features/Semaphores.md)
   - [Condition Variables](03-Kernel-Features/Condition-Variables.md)
   - [Events](03-Kernel-Features/Events.md)
   - [Queues](03-Kernel-Features/Queues.md)
   - [Notifications](03-Kernel-Features/Notifications.md)
   - [Timers](03-Kernel-Features/Timers.md)
   - [Sleep and Tick](03-Kernel-Features/Sleep-and-Tick.md)
   - [Other Primitives](03-Kernel-Features/Other-Primitives.md)
4. [Interrupts](04-Interrupts.md)
5. [Memory and Static Allocation](05-Memory-and-Static-Allocation.md)
6. [Architecture and Ports](06-Architecture-and-Ports.md)
7. [Supported Devices](Supported-Devices.md) — BSP repos & boards  
8. [Building and Examples](07-Building-and-Examples.md) · [Example catalog](07-Examples/index.md)
9. [C / C++ / Rust](08-C-CXX-Rust.md)
10. [API Reference](09-API-Reference/index.md) — one page per `include/rt/*.h`
11. [Porting Guide](10-Porting-Guide.md)
12. [FreeRTOS / Zephyr Comparison](11-FreeRTOS-Zephyr-Comparison.md)
13. [Benchmarks](12-Benchmarks.md) — what exists / what does not

## Quick facts

| Item | Value (from this tree) |
|------|------------------------|
| Version | 0.20.1 (`Cargo.toml`) |
| License | Apache-2.0 |
| Author | Chris Copeland \<chris@chrisnc.net\> |
| Build (host) | SCons + clang → `build/signal/` |
| Host sim port | `arch/signal` (POSIX signals; **not** a chip port) |
| Chip ports | Arm, AArch64, RISC-V, TI C28 (+ external BSPs) |
| Priority range | 0 … 31 (0 highest; 31 = idle) |
| Hart model | Largely single-hart ready-queue |

## Suggested reading order

1. Introduction → Getting Started → Tasks / Scheduling  
2. One sync chapter (Mutexes or Semaphores)  
3. Interrupts, then Architecture / Supported Devices  
4. Building / Examples, then API Reference while coding  
5. Benchmarks before drawing performance conclusions  
