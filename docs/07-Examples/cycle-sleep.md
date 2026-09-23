# Example: `cycle/sleep.c`

Sleep wake timing: one task sleeps 1 tick after stamping; peer samples on wake cadence.

## Cycle / trap micro-bench

These programs under `examples/cycle/` are **cycle / trap micro-benchmarks**, not functional pass/fail tests. They:

1. Record `rt_cycle()` timestamps around a primitive operation for `N` (100) iterations into `cycles[N]`.
2. Call `rt_trap()` so a debugger / host harness can inspect the samples (they do **not** call `rt_exit` with an assert suite).

Host builds typically enable `RT_CYCLE_ENABLE`. What is measured is wall-cycle delta on the observing task from a `start_cycle` stamped by its peer (or itself), i.e. wake / handoff latency including scheduling.


**This file measures:** Sleep / tick wake latency (1-tick sleeps).

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `sleep` | 0 | `RT_STACK_MIN` | Stamp `start_cycle`, `rt_task_sleep(1)`, × N. |
| `task1` | 1 | `RT_STACK_MIN` | Sample delta into `cycles[i]`, sleep(1), × N; then `rt_trap`. |

`task1` is higher priority so it runs the sample when both become ready after sleeps.

## Shared state

```c
#define N 100
static volatile uint32_t start_cycle = 0;
static volatile uint32_t cycles[N];
```

## Control flow

1. Lower-priority `sleep` stamps and sleeps one tick.
2. Higher-priority `task1` records how many cycles elapsed since that stamp (wake / tick path), then sleeps itself.
3. Alternating 1-tick sleeps produce N samples of sleep-related latency.

Measured quantity ≈ time from pre-sleep stamp to the peer’s observation after tick wakeups.

## APIs used

| API | Role |
|-----|------|
| `rt_cycle` | Timestamp. |
| `rt_task_sleep` | Operation under test. |
| `rt_trap` | End harness. |

## Success / failure

- **“Pass” for a micro-bench:** completes `N` samples and hits `rt_trap()` without crashing.
- There is **no** assert on cycle magnitudes here; analysis is offline from `cycles[]`.
- Not a CI golden-number test unless an external harness adds one.

## Build / run

Host signal port (cycle counters enabled in the default host SConstruct):

```bash
scons -j$(nproc)
./build/signal/examples/cycle/sleep
```

On bare metal, run under a debugger to catch `rt_trap` and dump `cycles`.

## See also

- [API: cycle](../09-API-Reference/cycle.md) · [API: trap](../09-API-Reference/trap.md) · [Benchmarks](../12-Benchmarks.md)
- [Sleep and Tick](../03-Kernel-Features/Sleep-and-Tick.md) · [sleep](sleep.md)
- [Building and Examples](../07-Building-and-Examples.md)
