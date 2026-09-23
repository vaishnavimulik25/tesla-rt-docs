# Example: `cycle/yield.c`

Equal-priority yield ping-pong latency via `rt_cycle` / `rt_trap`.

## Cycle / trap micro-bench

These programs under `examples/cycle/` are **cycle / trap micro-benchmarks**, not functional pass/fail tests. They:

1. Record `rt_cycle()` timestamps around a primitive operation for `N` (100) iterations into `cycles[N]`.
2. Call `rt_trap()` so a debugger / host harness can inspect the samples (they do **not** call `rt_exit` with an assert suite).

Host builds typically enable `RT_CYCLE_ENABLE`. What is measured is wall-cycle delta on the observing task from a `start_cycle` stamped by its peer (or itself), i.e. wake / handoff latency including scheduling.


**This file measures:** Yield-to-peer latency between two priority-0 tasks.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `task1` | 0 | `RT_STACK_MIN` | On each iter: sample `rt_cycle() - start_cycle`, then yield. Ends with `rt_trap`. |
| `task0` | 0 | `RT_STACK_MIN` | On each iter: stamp `start_cycle`, then yield. |

Source note: equal-priority tasks may start in constructor registration order — `task1` is declared before `task0` so the sampler can run first after the peer stamps.

## Shared state

```c
#define N 100
static volatile uint32_t start_cycle = 0;
static volatile uint32_t cycles[N];
```

## Control flow

1. `task0` stores `start_cycle = rt_cycle()` then `rt_task_yield()`.
2. `task1` runs, stores delta into `cycles[i]`, yields back.
3. After N rounds, `task1` calls `rt_trap()`.

Measured quantity ≈ one yield handoff (plus any port overhead between stamp and sample).

## APIs used

| API | Role |
|-----|------|
| `rt_cycle` | Read cycle counter. |
| `rt_task_yield` | Operation under test. |
| `rt_trap` | Stop for inspection. |

## Success / failure

- **“Pass” for a micro-bench:** completes `N` samples and hits `rt_trap()` without crashing.
- There is **no** assert on cycle magnitudes here; analysis is offline from `cycles[]`.
- Not a CI golden-number test unless an external harness adds one.

## Build / run

Host signal port (cycle counters enabled in the default host SConstruct):

```bash
scons -j$(nproc)
./build/signal/examples/cycle/yield
```

On bare metal, run under a debugger to catch `rt_trap` and dump `cycles`.

## See also

- [API: cycle](../09-API-Reference/cycle.md) · [API: trap](../09-API-Reference/trap.md) · [Benchmarks](../12-Benchmarks.md)
- [Scheduling](../03-Kernel-Features/Scheduling.md) · [simple](simple.md) · sibling [cycle-sleep](cycle-sleep.md)
- [Building and Examples](../07-Building-and-Examples.md)
