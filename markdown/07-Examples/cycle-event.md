# Example: `cycle/event.c`

Event set→wait wake latency for a single bit.

## Cycle / trap micro-bench

These programs under `examples/cycle/` are **cycle / trap micro-benchmarks**, not functional pass/fail tests. They:

1. Record `rt_cycle()` timestamps around a primitive operation for `N` (100) iterations into `cycles[N]`.
2. Call `rt_trap()` so a debugger / host harness can inspect the samples (they do **not** call `rt_exit` with an assert suite).

Host builds typically enable `RT_CYCLE_ENABLE`. What is measured is wall-cycle delta on the observing task from a `start_cycle` stamped by its peer (or itself), i.e. wake / handoff latency including scheduling.


**This file measures:** Event set→wait latency.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `setter` | 1 | `RT_STACK_MIN` | Stamp, `rt_event_set(&event, 1)`, × N. |
| `waiter` | 0 | `RT_STACK_MIN` | `rt_event_wait(&event, 1)`, sample, × N; `rt_trap`. |

## Shared state

```c
static RT_EVENT(event);
static volatile uint32_t start_cycle, cycles[N];
```

## Control flow

Classic producer stamp + set, consumer wait + sample. Measured: event bit set→wait latency.

## APIs used

| API | Role |
|-----|------|
| `RT_EVENT` / `rt_event_set` / `rt_event_wait` | Primitive under test. |
| `rt_cycle` / `rt_trap` | Instrumentation. |

## Success / failure

- **“Pass” for a micro-bench:** completes `N` samples and hits `rt_trap()` without crashing.
- There is **no** assert on cycle magnitudes here; analysis is offline from `cycles[]`.
- Not a CI golden-number test unless an external harness adds one.

## Build / run

Host signal port (cycle counters enabled in the default host SConstruct):

```bash
scons -j$(nproc)
./build/signal/examples/cycle/event
```

On bare metal, run under a debugger to catch `rt_trap` and dump `cycles`.

## See also

- [API: cycle](../09-API-Reference/cycle.md) · [API: trap](../09-API-Reference/trap.md) · [Benchmarks](../12-Benchmarks.md)
- [Events](../03-Kernel-Features/Events.md) · [event](event.md)
- [Building and Examples](../07-Building-and-Examples.md)
