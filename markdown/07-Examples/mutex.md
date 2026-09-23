# Example: `mutex.c`

Three equal-priority tasks increment a shared counter using lock, trylock+adopt, and timedlock+adopt paths; a low-priority timeout panics if they stall.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `increment_lock` | 1 | `RT_STACK_MIN` | `RT_MUTEX_GUARD` each of `ITERATIONS` increments. |
| `increment_trylock` | 1 | `RT_STACK_MIN` | Spin `rt_mutex_trylock` + sleep(1), then `RT_MUTEX_GUARD_ADOPT`. |
| `increment_timedlock` | 1 | `RT_STACK_MIN` | Loop `rt_mutex_timedlock(..., 1)` until success, then adopt. |
| `timeout` | 0 | `RT_STACK_MIN` | Sleeps 1000 ticks; `rt_panic("timed out")` if still running. |

`ITERATIONS = 10000`, `NUM_TASKS = 3`.

## Shared state

```c
static RT_MUTEX(mutex);
static unsigned x = 0;
/* exit_last uses static RT_SEM(exit_sem, NUM_TASKS - 1) */
```

The exit semaphore starts at `NUM_TASKS - 1` so the first two finishers consume a count via `trywait`; the last finisher fails `trywait` and runs the assert + `rt_exit`.

## Control flow

1. All three workers race to take `mutex`, each performing 10 000 increments of `x`.
2. `increment_lock` uses RAII-style `RT_MUTEX_GUARD` (cleanup attribute unlocks on scope exit).
3. The trylock / timedlock workers take the mutex manually, then wrap it in `RT_MUTEX_GUARD_ADOPT` so unlock still happens via cleanup.
4. After its loop, each worker calls `exit_last()`:
   - If `rt_sem_trywait` succeeds → another task still outstanding → return.
   - If it fails → last worker → assert `x == ITERATIONS * NUM_TASKS` → `rt_exit`.
5. If workers hang, priority-0 `timeout` panics after 1000 ticks.

Equal priority + sleep in the trylock path keeps the system progressing without priority donation being the focus (see [donate](donate.md) for that).

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_MUTEX` | Shared mutual exclusion. |
| `RT_MUTEX_GUARD` | Scoped lock for the plain lock path. |
| `rt_mutex_trylock` / `rt_mutex_timedlock` | Non-blocking / timed acquisition. |
| `RT_MUTEX_GUARD_ADOPT` | Adopt an already-held mutex into a guard. |
| `RT_SEM` + `rt_sem_trywait` | “Last task exits” barrier. |
| `rt_assert` | Verify final `x`. |
| `rt_panic` | Watchdog failure path. |

## Success / failure

- **Pass:** `x == 30000`, last worker `rt_exit`.
- **Fail:** wrong `x` (lost updates / double unlock bugs), or `rt_panic("timed out")`.

> Skipped when building with `cl2000` (no `__attribute__((cleanup))`).

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/mutex
```

## See also

- [Mutexes](../03-Kernel-Features/Mutexes.md)
- [fair](fair.md) · [recursive](recursive.md) · [donate](donate.md) · [cycle-mutex](cycle-mutex.md)
- [Building and Examples](../07-Building-and-Examples.md)
