# Examples Index

All first-party examples live in the core `rt` repository under `examples/`. Each page below is a full source walkthrough (tasks, shared state, control flow, APIs, pass/fail, build). Host binaries: `./build/signal/examples/...` after `scons`.

## Functional examples

| Page | Blurb |
|------|-------|
| [empty](empty.md) | Smallest smoke test: one task drops privilege and `rt_exit`s. |
| [simple](simple.md) | Two equal-priority arg-tasks yield 100×; `arg==1` exits. |
| [sem](sem.md) | Poster/waiter counting semaphore with intentional timed-wait failure. |
| [mutex](mutex.md) | Three lock styles (guard / trylock / timedlock) on one counter + last-exit sem. |
| [recursive](recursive.md) | Nested `foo`→`bar` locking on `RT_MUTEX_RECURSIVE`. |
| [donate](donate.md) | Priority inversion + donation; strict `sequence()` order 0…10. |
| [fair](fair.md) | Four equal-priority tasks assert mutex handoff fairness. |
| [cond](cond.md) | Signaler + two incrementers on `RT_COND` / predicate `flag`. |
| [event](event.md) | Event bits with ALL vs ANY+NOCLEAR waits and a final timeout. |
| [notify](notify.md) | Notify OR/clear-wait loop mirroring the sem timing pattern. |
| [queue](queue.md) | Dual pushers + peek/pop ordering stress until duration exit. |
| [rwlock](rwlock.md) | Readers assert `x==y` while a writer bumps both under write lock. |
| [once](once.md) | Contended `rt_once_call` with deliberate `once.done` reset. |
| [pool](pool.md) | Pool get → queue → put pipeline with back-pressure. |
| [timer](timer.md) | Periodic software timer start/stop/change_period count checks. |
| [sleep](sleep.md) | `rt_task_sleep_periodic` wake skew ≤1 tick for two periods. |
| [tls](tls.md) | Per-task `foo`/`bar` isolation via TLS or split statics. |
| [float](float.md) | FP accumulate across yields with `RT_STACK_FP_MIN`. |

## Cycle micro-benchmarks

Instrumentation harnesses: stamp `rt_cycle()`, exercise a primitive `N=100` times, store deltas, `rt_trap()` for inspection (not assert-based pass/fail).

| Page | Measures |
|------|----------|
| [cycle-yield](cycle-yield.md) | Equal-priority yield handoff latency. |
| [cycle-sleep](cycle-sleep.md) | 1-tick sleep wake timing. |
| [cycle-sem](cycle-sem.md) | Sem post→wait latency. |
| [cycle-mutex](cycle-mutex.md) | Contended mutex unlock→lock latency. |
| [cycle-event](cycle-event.md) | Event set→wait latency. |
| [cycle-notify](cycle-notify.md) | Notify post→wait latency. |
| [cycle-queue](cycle-queue.md) | Queue push→pop latency. |

## Water synchronization lab

Shared harness [`water/water.c`](water-water.md) + `water.h` (`hydrogen` / `oxygen` / `make_water`). Two H loops (prio 2), one O loop (prio 1), timeout supervisor (1000 ticks) checks bond counts. Variants only change the rendezvous implementation:

| Page | Strategy |
|------|----------|
| [water-sem](water-sem.md) | Atomic H pairing + `h2ready` / `hdone` semaphores. |
| [water-cond](water-cond.md) | Mutex + `hready` / `hdone` condition variables. |
| [water-barrier](water-barrier.md) | Capacity sems (2 H, 1 O) + 3-party barrier around `make_water`. |
| [water-harness](water-water.md) | Shared loops, counters, and success criteria. |

See also [Building and Examples](../07-Building-and-Examples.md) and [Benchmarks](../12-Benchmarks.md).
