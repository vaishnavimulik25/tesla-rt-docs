# FreeRTOS / Zephyr Comparison

Honest, talk-oriented comparison of **open-source Tesla RT (`rt`) 0.20.1** vs typical FreeRTOS / Zephyr properties. FreeRTOS/Zephyr columns are generalized; always check your exact version.

| Topic | Tesla RT (`rt`) | FreeRTOS (typical) | Zephyr (typical) |
|-------|-----------------|--------------------|------------------|
| License | Apache-2.0 | MIT | Apache-2.0 |
| Preemption | Full preemption; priority 0 = highest | Preemptive; lower number = higher prio (common config) | Preemptive; lower number = higher prio |
| Priority range | 0..31 (31 = idle) | Config `configMAX_PRIORITIES` | Configurable |
| Sync: mutex | Yes + **priority donation** + recursive | Yes + priority inheritance (opt) | Yes + priority inheritance |
| Sync: semaphore | Counting + binary; ISR post helpers | Counting + binary | Wide set (k_sem, …) |
| Condvar | Yes (`rt_cond`) | Not classic (use task notify / queues) | `k_condvar` |
| Event bits | Yes (`rt_event`) | Event groups | `k_event` |
| Queue | Bounded **lock-free MPMC** | Queue / stream / message buffers | `k_queue`, `k_msgq`, … |
| Notifications | Standalone `rt_notify` object | Per-task notifications | Poll / events / futex-like patterns |
| Timers | Software timers via daemon task + queue | Timer service task | `k_timer` |
| RW lock / barrier / once / pool | Yes in-tree | Mostly via add-ons / app | Many in libc/kernel |
| Static allocation | First-class macros (`RT_TASK`, `RT_*`) | Static alloc APIs + heap option | Static + heap slabs/pools |
| Heap | No primary public heap API in headers | `pvPortMalloc` common | `k_malloc` / heaps |
| ISR safety | Nonblocking + pendable syscalls; no IRQ-disable sync when HW atomics exist | FromISR APIs; critical sections common | IRQ-safe APIs; spinlocks on SMP |
| Critical sections | Avoided for sync by design (atomics) | `taskENTER_CRITICAL` widely used | Spinlocks / irq_lock |
| SMP | Stock tree **largely single-hart** | SMP variants exist | First-class SMP |
| Languages | C, C++ wrappers, Rust crate | C (+ third-party) | C, C++ |
| Host sim | `arch/signal` POSIX | POSIX/Windows ports / FreeRTOS-Sim | Native sim / QEMU |
| Trace | Optional `RT_TRACE_ENABLE` | Trace macros / Percepio etc. | Tracing subsystem |

## Talking points

1. **`rt`’s differentiator** is atomics-first sync and ISR-capable nonblocking posts without making IRQ-disable the default locking story.
2. **Priority donation** for mutexes is implemented in-tree (`examples/donate.c`) — call it out vs “inheritance optional” in other RTOSes.
3. **Queue design** (lock-free MPMC bounded) is closer to concurrent data-structure practice than a classical single-lock RTOS queue.
4. **Scope**: `rt` is a kernel + ports, not a full OS (Zephyr’s breadth) or an ecosystem of middleware (FreeRTOS+TCP, …).
5. Do **not** equate this open-source tree with a closed vehicle OS SKU when presenting.

## When to choose which (opinionated, for discussion)

- Need tiny, preemptible kernel with strong C11 atomics story → **`rt`**
- Need huge ecosystem / many app notes → **FreeRTOS**
- Need full OS services, device model, SMP products → **Zephyr**
