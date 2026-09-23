# API Reference Index

Application-facing headers live in `include/rt/*.h`. Pages below list public types, macros, and functions with blocking/ISR notes. Symbols used only for ports, tracing, or intrusive list plumbing are marked **internal**.

## Tasks and scheduling

| Header | Page |
|--------|------|
| `task.h` | [task](task.md) |
| `start.h` | [start](start.md) |
| `stack.h` | [stack](stack.md) |
| `tls.h` | [tls](tls.md) |
| `idle.h` | [idle](idle.md) |
| `context.h` | [context](context.md) (advanced / port) |

## Synchronization

| Header | Page |
|--------|------|
| `mutex.h` | [mutex](mutex.md) |
| `sem.h` | [sem](sem.md) |
| `cond.h` | [cond](cond.md) |
| `event.h` | [event](event.md) |
| `queue.h` | [queue](queue.md) |
| `notify.h` | [notify](notify.md) |
| `barrier.h` | [barrier](barrier.md) |
| `rwlock.h` | [rwlock](rwlock.md) |
| `once.h` | [once](once.md) |

## Time

| Header | Page |
|--------|------|
| `tick.h` | [tick](tick.md) |
| `timer.h` | [timer](timer.md) |
| `cycle.h` | [cycle](cycle.md) |

## Memory / pools

| Header | Page |
|--------|------|
| `pool.h` | [pool](pool.md) |
| `mpu.h` | [mpu](mpu.md) |

## Interrupts and deferred work

| Header | Page |
|--------|------|
| `interrupt.h` | [interrupt](interrupt.md) |
| `pend_function.h` | [pend_function](pend_function.md) |
| `syscall.h` | [syscall](syscall.md) (**internal / port**) |

## Diagnostics and control

| Header | Page |
|--------|------|
| `log.h` | [log](log.md) |
| `trace.h` | [trace](trace.md) (hooks; advanced) |
| `assert.h` | [assert](assert.md) |
| `panic.h` | [panic](panic.md) |
| `abort.h` | [abort](abort.md) |
| `exit.h` | [exit](exit.md) |
| `trap.h` | [trap](trap.md) |

## Other

| Header | Page |
|--------|------|
| `atomic.h` | [atomic](atomic.md) |
| `list.h` | [list](list.md) (**internal**) |
| `container.h` | [container](container.md) (**internal**) |

Approximate public surface documented from headers: **~160 functions** and **~110 macros** (including architecture/config macros). Prefer the Kernel Features chapters for conceptual guidance.
