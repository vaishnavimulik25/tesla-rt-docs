# Sleep and Tick

## Tick API (`rt/tick.h`)

| Function | Description |
|----------|-------------|
| `rt_tick_advance()` | Advance to next tick; call periodically from timer IRQ / signal |
| `rt_tick_count()` | Current tick (`unsigned long`) |
| `rt_tick_elapse(ticks, start_tick)` | Inline helper to subtract elapsed time from a remaining budget |

Tick period / Hz is **board/port specific** — not fixed in the portable headers (**not documented in-tree** as a global constant).

## Sleep API (`rt/task.h` + `src/sleep.c`)

```c
void rt_task_sleep(unsigned long ticks);  // ticks==0 → return
void rt_task_sleep_periodic(unsigned long *last_wake_tick, unsigned long period);
```

Periodic sleep:

- Computes next wake as `*last_wake_tick + period` (updates `*last_wake_tick` first).
- `period == 0` → no-op.
- If the deadline already passed, the syscall path can keep the task ready (see comments in `src/rt.c`).

## Example

`examples/sleep.c` — two tasks with different periods via `RT_TASK_ARG`.

## Pendable tick

`RT_SYSCALL_PENDABLE_TICK` exists for deferred tick processing from interrupt context (see `rt/syscall.h`).
