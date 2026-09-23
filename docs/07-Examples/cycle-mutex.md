# Example: `cycle/mutex.c`

Contended mutex unlock→lock handoff latency (holder sleeps while owning, then releases; waiter acquires and samples).

## Cycle / trap micro-bench

These programs under `examples/cycle/` are **cycle / trap micro-benchmarks**, not functional pass/fail tests. They:

1. Record `rt_cycle()` timestamps around a primitive operation for `N` (100) iterations into `cycles[N]`.
2. Call `rt_trap()` so a debugger / host harness can inspect the samples (they do **not** call `rt_exit` with an assert suite).

Host builds typically enable `RT_CYCLE_ENABLE`. What is measured is wall-cycle delta on the observing task from a `start_cycle` stamped by its peer (or itself), i.e. wake / handoff latency including scheduling.


**This file measures:** Contended mutex handoff latency.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `task0` | 1 | `RT_STACK_MIN` | Lock, sleep(2), stamp `start_cycle` just before unlock (guard end), × N. |
| `task1` | 0 | `RT_STACK_MIN` | Sleep(1), lock, sample delta, × N; `rt_trap`. |

## Shared state

```c
static RT_MUTEX(mutex);
static volatile uint32_t start_cycle, cycles[N];
```

## Control flow

1. `task1` sleeps 1 so `task0` can take the mutex first.
2. `task0` holds the lock across `rt_task_sleep(2)`, then stamps immediately before the guard unlocks.
3. `task1` acquires and records the handoff latency.
4. Pattern repeats N times.

Measured: contended mutex release→acquire path.

> Skipped for `cl2000` (cleanup guards).

## APIs used

| API | Role |
|-----|------|
| `RT_MUTEX` / `RT_MUTEX_GUARD` | Contended lock under test. |
| `rt_task_sleep` | Create overlap / waiter. |
| `rt_cycle` / `rt_trap` | Instrumentation. |

## Success / failure

- **“Pass” for a micro-bench:** completes `N` samples and hits `rt_trap()` without crashing.
- There is **no** assert on cycle magnitudes here; analysis is offline from `cycles[]`.
- Not a CI golden-number test unless an external harness adds one.

## Build / run

Host signal port (cycle counters enabled in the default host SConstruct):

```bash
scons -j$(nproc)
./build/signal/examples/cycle/mutex
```

On bare metal, run under a debugger to catch `rt_trap` and dump `cycles`.

## See also

- [API: cycle](../09-API-Reference/cycle.md) · [API: trap](../09-API-Reference/trap.md) · [Benchmarks](../12-Benchmarks.md)
- [Mutexes](../03-Kernel-Features/Mutexes.md) · [mutex](mutex.md)
- [Building and Examples](../07-Building-and-Examples.md)
