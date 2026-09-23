# Memory and Static Allocation

`rt` is oriented around **compile-time / link-time** allocation. Dynamic heap allocation of kernel objects is not the primary model in the public headers.

## Task stacks

```c
#include <rt/stack.h>

RT_STACK(name, size);           // aligned char array in .stack.* section
RT_STACKS(name, size, num);     // array of stacks
```

`RT_TASK` / `RT_TASK_ARG` create a private `RT_STACK(fn_task_stack, stack_size_)` and pass it to `rt_context_init`.

Minimum stack size `RT_STACK_MIN` is **architecture-defined** (`rt/arch/stack.h`):

| Port | Typical `RT_STACK_MIN` (from headers) |
|------|----------------------------------------|
| `arch/signal` | `1UL` (host; context is signal/user-stack oriented) |
| Arm | 128 or 512 depending on profile/config |
| AArch64 | 1024UL |
| RISC-V | 1024UL |
| C28 | 128 / 192 / 256 depending on config |

Always size stacks for your call depth, FPU lazy context, and ISR nesting on the target.

## Static sync objects

Macros declare and initialize BSS/data objects:

| Macro | Header |
|-------|--------|
| `RT_MUTEX` / `RT_MUTEX_RECURSIVE` | `mutex.h` |
| `RT_SEM` / `RT_SEM_BINARY` / `RT_SEM_MAX` | `sem.h` |
| `RT_COND` | `cond.h` |
| `RT_EVENT` | `event.h` |
| `RT_NOTIFY` | `notify.h` |
| `RT_QUEUE` / `RT_QUEUE_STATIC` | `queue.h` |
| `RT_RWLOCK` | `rwlock.h` |
| `RT_BARRIER` | `barrier.h` |
| `RT_ONCE` | `once.h` |
| `RT_POOL` / `RT_POOL_STATIC` | `pool.h` |
| `RT_TIMER_*` + `RT_TIMER_QUEUE(_STATIC)` | `timer.h` |

Queues and pools allocate backing arrays as static storage next to the object.

## Task control blocks

`RT_TASK` / `RT_TASK_ARG` emit a static `struct rt_task` (optionally in a privileged section via `RT_MPU_PRIV_DATA`). `rt_task_init` may only be called **before** `rt_start`.

## MPU / privilege (optional)

Compile with `-DRT_MPU_TASK_REGIONS_ENABLE=1` (Cargo feature `task-mpu`) to attach per-task MPU regions. Extra `RT_TASK(..., region...)` arguments become MPU regions while the task runs. Stack (and TLS, if enabled) regions are wired automatically.

Privileged section helpers (`rt/mpu.h`):

- `RT_MPU_PRIV_DATA(name)` / `RT_MPU_PRIV_BSS(name)` — when `RT_MPU_PRIV_SECTIONS_ENABLE`

`rt_task_drop_privilege()` makes the current task unprivileged where the architecture supports it (no-op on some ports).

## Thread-local storage (optional)

`-DRT_TASK_LOCAL_STORAGE_ENABLE=1` with size `RT_TASK_LOCAL_STORAGE_SIZE` (default 32). Scheduler calls `rt_tls_set` on context switch. See `rt/tls.h`.

## Cycle accounting (optional)

`-DRT_CYCLE_ENABLE=1` and optionally `-DRT_TASK_CYCLE_ENABLE=1` (requires cycle enable). Host `SConstruct` turns both on for examples.

## What is not provided in-tree

- A general-purpose kernel heap allocator API is **not documented** as a first-class public feature in `include/rt/*.h`.
- Memory pools (`rt_pool_get` / `rt_pool_put`) manage a fixed array of object pointers — not a byte heap.
