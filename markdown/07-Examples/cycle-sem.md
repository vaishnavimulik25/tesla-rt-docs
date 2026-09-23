# Example: `cycle/sem.c`

Semaphore post→wait handoff latency.

## Cycle / trap micro-bench

These programs under `examples/cycle/` are **cycle / trap micro-benchmarks**, not functional pass/fail tests. They:

1. Record `rt_cycle()` timestamps around a primitive operation for `N` (100) iterations into `cycles[N]`.
2. Call `rt_trap()` so a debugger / host harness can inspect the samples (they do **not** call `rt_exit` with an assert suite).

Host builds typically enable `RT_CYCLE_ENABLE`. What is measured is wall-cycle delta on the observing task from a `start_cycle` stamped by its peer (or itself), i.e. wake / handoff latency including scheduling.


**This file measures:** Counting-semaphore post→wait latency.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `poster` | 1 | `RT_STACK_MIN` | Stamp, `rt_sem_post`, × N. |
| `waiter` | 0 | `RT_STACK_MIN` | `rt_sem_wait`, sample delta, × N; `rt_trap`. |

## Shared state

```c
static RT_SEM(sem, 0);
static volatile uint32_t start_cycle, cycles[N];
```

## Control flow

1. Waiter blocks on empty sem.
2. Poster (higher prio) stamps and posts — wakes waiter.
3. Waiter records cycles from stamp to post-wake.
4. Repeat N times; trap.

Measured: post→wait wake latency including schedule.

## APIs used

| API | Role |
|-----|------|
| `RT_SEM` / `rt_sem_post` / `rt_sem_wait` | Primitive under test. |
| `rt_cycle` / `rt_trap` | Instrumentation. |

## Success / failure

- **“Pass” for a micro-bench:** completes `N` samples and hits `rt_trap()` without crashing.
- There is **no** assert on cycle magnitudes here; analysis is offline from `cycles[]`.
- Not a CI golden-number test unless an external harness adds one.

## Build / run

Host signal port (cycle counters enabled in the default host SConstruct):

```bash
scons -j$(nproc)
./build/signal/examples/cycle/sem
```

On bare metal, run under a debugger to catch `rt_trap` and dump `cycles`.

## See also

- [API: cycle](../09-API-Reference/cycle.md) · [API: trap](../09-API-Reference/trap.md) · [Benchmarks](../12-Benchmarks.md)
- [Semaphores](../03-Kernel-Features/Semaphores.md) · [sem](sem.md)
- [Building and Examples](../07-Building-and-Examples.md)
