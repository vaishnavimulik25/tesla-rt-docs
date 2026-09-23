# Example: `cycle/queue.c`

Queue push→pop handoff latency.

## Cycle / trap micro-bench

These programs under `examples/cycle/` are **cycle / trap micro-benchmarks**, not functional pass/fail tests. They:

1. Record `rt_cycle()` timestamps around a primitive operation for `N` (100) iterations into `cycles[N]`.
2. Call `rt_trap()` so a debugger / host harness can inspect the samples (they do **not** call `rt_exit` with an assert suite).

Host builds typically enable `RT_CYCLE_ENABLE`. What is measured is wall-cycle delta on the observing task from a `start_cycle` stamped by its peer (or itself), i.e. wake / handoff latency including scheduling.


**This file measures:** Queue push→pop latency.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `popper` | 0 | `RT_STACK_MIN` | `rt_queue_pop`, sample delta, × N; `rt_trap`. |
| `pusher` | 1 | `RT_STACK_MIN` | Stamp, `rt_queue_push`, × N. |

Popper is declared first at lower priority so it can block empty before pushes land.

## Shared state

```c
RT_QUEUE_STATIC(queue, int, 10);
static volatile uint32_t start_cycle, cycles[N];
```

## Control flow

1. Popper blocks on empty queue.
2. Pusher stamps and pushes — wakes popper.
3. Popper records latency.
4. Depth 10 avoids filler saturation for N=100 with 1:1 pacing.

Measured: push→pop wake latency.

## APIs used

| API | Role |
|-----|------|
| `RT_QUEUE_STATIC` / `rt_queue_push` / `rt_queue_pop` | Primitive under test. |
| `rt_cycle` / `rt_trap` | Instrumentation. |

## Success / failure

- **“Pass” for a micro-bench:** completes `N` samples and hits `rt_trap()` without crashing.
- There is **no** assert on cycle magnitudes here; analysis is offline from `cycles[]`.
- Not a CI golden-number test unless an external harness adds one.

## Build / run

Host signal port (cycle counters enabled in the default host SConstruct):

```bash
scons -j$(nproc)
./build/signal/examples/cycle/queue
```

On bare metal, run under a debugger to catch `rt_trap` and dump `cycles`.

## See also

- [API: cycle](../09-API-Reference/cycle.md) · [API: trap](../09-API-Reference/trap.md) · [Benchmarks](../12-Benchmarks.md)
- [Queues](../03-Kernel-Features/Queues.md) · [queue](queue.md)
- [Building and Examples](../07-Building-and-Examples.md)
