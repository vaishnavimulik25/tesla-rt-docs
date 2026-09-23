# Example: `fair.c`

Four equal-priority tasks share a mutex and assert that no waiter is starved past the current max progress counter — a fairness check on mutex handoff.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `task` ×4 (`RT_TASK_ARG` indices 0…3) | 1 | `2 * RT_STACK_MIN` | Each does `ITERATIONS` (100) guarded increments of `x[task_index]`. |
| `timeout` | 0 | `RT_STACK_MIN` | Sleeps 1000; `rt_panic("timed out")`. |

## Shared state

```c
static RT_MUTEX(mutex);
static uint32_t x[NUM_TASKS];
static uint32_t max_value;
/* exit_last: static RT_SEM(exit_sem, NUM_TASKS - 1) */
```

## Control flow

1. Each worker, under `RT_MUTEX_GUARD`:
   - `my_x = ++x[task_index]`
   - assert `max_value <= my_x` — if another task raced far ahead while this one waited unfairly, `my_x` could lag `max_value` and fail.
   - `max_value = my_x`
   - `rt_task_sleep(1)` still holding? No — sleep is **inside** the guard in source:

```c
RT_MUTEX_GUARD(guard, mutex);
const uint32_t my_x = ++x[task_index];
rt_assert(max_value <= my_x, "the mutex is unfair");
max_value = my_x;
rt_task_sleep(1);
```

   Sleeping while holding the mutex forces contention and makes unfair wake order visible: after unlock, the next owner should be a waiter that is not chronically skipped.
2. `exit_last()` uses the same “last of N” semaphore pattern as `mutex.c`.
3. Last task asserts every `x[i] == ITERATIONS` and `rt_exit`.

Equal priorities are essential: fairness among equal-priority waiters is what is under test.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_TASK_ARG` | Four instances with distinct indices. |
| `RT_MUTEX_GUARD` | Critical section around progress update. |
| `rt_task_sleep` | Hold lock across a tick to amplify contention. |
| `RT_SEM` / `trywait` | Coordinate exit. |
| `rt_panic` | Watchdog. |

## Success / failure

- **Pass:** all four complete 100 iterations without fairness assert; `rt_exit`.
- **Fail:** `"the mutex is unfair"`, `"a task did not run enough"`, or timeout panic.

> Skipped for `cl2000`.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/fair
```

## See also

- [Mutexes](../03-Kernel-Features/Mutexes.md) · [Scheduling](../03-Kernel-Features/Scheduling.md)
- [mutex](mutex.md) · [donate](donate.md)
- [Building and Examples](../07-Building-and-Examples.md)
