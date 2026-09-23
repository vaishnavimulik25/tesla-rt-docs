# Example: `sleep.c`

Two tasks use `rt_task_sleep_periodic` with different periods and assert wake skew is at most one tick; last finisher exits.

## Tasks / roles

| Task | Arg (period) | Priority | Stack | Role |
|------|--------------|----------|-------|------|
| `sleep_periodic` | 10 | 0 | `RT_STACK_MIN` | 5 periodic sleeps of period 10. |
| `sleep_periodic` | 20 | 1 | `RT_STACK_MIN` | 5 periodic sleeps of period 20. |

`nloops = 5`. Higher priority on the longer period task is incidental scheduling preference between them.

## Shared state

```c
/* per-task: unsigned long last_wake_tick = 0; */
static RT_SEM(exit_sem, 1);  /* inside sleep_periodic, static — last exit wins */
```

## Control flow

1. Each call `rt_task_sleep_periodic(&last_wake_tick, period)` updates the anchor and sleeps until the next absolute period boundary.
2. After wake, `now = rt_tick_count()`; assert `(now - last_wake_tick) <= 1` — wake should land on the intended tick (≤1 tick of scheduling latency).
3. After 5 loops, `exit_last`-style: `trywait` on `exit_sem` initial count 1; first finisher consumes it; second fails trywait and `rt_exit`s.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `rt_task_sleep_periodic` | Absolute/periodic sleep helper. |
| `rt_tick_count` | Observe current tick for skew check. |
| `RT_SEM` / `trywait` | Dual-task exit coordination. |
| `RT_TASK_ARG` | Different periods per instance. |

## Success / failure

- **Pass:** all wake asserts hold; second task `rt_exit`.
- **Fail:** `"woke up at the wrong tick"`.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/sleep
```

## See also

- [Sleep and Tick](../03-Kernel-Features/Sleep-and-Tick.md)
- [timer](timer.md) · [cycle-sleep](cycle-sleep.md)
- [Building and Examples](../07-Building-and-Examples.md)
