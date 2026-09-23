# Benchmarks and Performance Evaluation

## What exists in-tree

### Cycle micro-benchmarks (`examples/cycle/`)

These programs measure **CPU cycle deltas** with `rt_cycle()` around a primitive operation, store N samples, then call `rt_trap()` for inspection:

| Program | Operation timed |
|---------|-----------------|
| `cycle/yield.c` | `rt_task_yield` between equal-priority tasks |
| `cycle/sleep.c` | `rt_task_sleep(1)` wake path |
| `cycle/sem.c` | `rt_sem_post` → `rt_sem_wait` |
| `cycle/mutex.c` | contended mutex acquire handoff |
| `cycle/event.c` | `rt_event_set` → `rt_event_wait` |
| `cycle/notify.c` | `rt_notify_post` → `rt_notify_wait` |
| `cycle/queue.c` | `rt_queue_push` → `rt_queue_pop` |

Host `SConstruct` defines `RT_CYCLE_ENABLE=1` and `RT_TASK_CYCLE_ENABLE=1` so these can run on `arch/signal`.

**How to use:** build the cycle programs, run under a debugger or semihosting environment, dump the `cycles[]` array after `rt_trap`. There is **no** automated reporter that prints averages in the core repo.

### Task cycle accounting

When `RT_TASK_CYCLE_ENABLE` is on, `struct rt_task` tracks `total_cycles` / `start_cycle`, with `rt_task_cycle_pause` / `rt_task_cycle_resume` for syscall accounting. This is infrastructure for measurement, not a published benchmark suite.

### Functional stress / fairness demos

| Example | Relevance |
|---------|-----------|
| `fair.c` | Equal-priority mutex fairness (correctness, not latency) |
| `donate.c` | Priority donation correctness under inversion |
| `water/*` | Sync composition under load until a tick timeout |

### QEMU runs

`test.bash <core>` runs examples under QEMU with a wall-clock `timeout`. That validates **liveness**, not interrupt latency charts.

## What does **not** exist (in the open trees reviewed)

- No checked-in **latency PDF/HTML report** or CI dashboard for context-switch times.
- No formal **interrupt latency** benchmark harness with published numbers.
- No `README` claims of “X cycles mean mutex handoff” as product guarantees.
- BSP repos may contain timing experiments locally; none were documented as a standard suite in the READMEs fetched for stm32/hercules/sitara/delfino/c29/esp32.

## Recommended evaluation method

1. Enable cycle counting on your port (`RT_CYCLE_ENABLE`).
2. Run the relevant `examples/cycle/*.c` (or BSP equivalents).
3. Capture `cycles[i]` after `rt_trap` / halt.
4. Report **your** board, compiler, optimization, and tick configuration — do not generalize to “Tesla RTOS” product claims.

## See also

- [Building and Examples](07-Building-and-Examples.md)
- [Examples catalog](07-Examples/index.md)
- [API: cycle.h](09-API-Reference/cycle.md)
