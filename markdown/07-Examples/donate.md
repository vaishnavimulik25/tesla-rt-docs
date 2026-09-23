# Example: `donate.c`

Priority-inversion scenario with nested mutexes and an explicit `sequence()` checker. With priority donation, the observed order is the asserted 0…10 sequence; without it, the mid-priority spinner would starve the high-priority lock holder.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `locker0` | 3 (highest) | `RT_STACK_MIN` | Takes `mutex0` then `mutex1`; records seq 3,5. |
| `locker1` | 2 | `RT_STACK_MIN` | Takes `mutex1`, sleeps, takes `mutex2`; records 2,4,8. |
| `spinner` | 1 | `RT_STACK_MIN` | Records 1, sleeps, then infinite `rt_task_yield` (CPU hog at mid priority). |
| `donator` | 0 (lowest) | `RT_STACK_MIN` | Holds `mutex0` early path; later tries `timedlock` on `mutex2`; records 0,6,7,9,10 and exits. |

## Shared state

```c
static rt_atomic_int seq = 0;
static RT_MUTEX(mutex0);
static RT_MUTEX(mutex1);
static RT_MUTEX(mutex2);
#define MAX_SEQ 10
```

`sequence(s)` asserts the next expected value of `seq` equals `s`, then increments. Reaching `MAX_SEQ` triggers `rt_exit`. Passing `-1` marks “should never run again” paths after unlocks (those asserts fire if a finished task runs unexpectedly).

## Control flow (why donation matters)

Intended order of `sequence` checkpoints: **0 → 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10**.

Rough narrative:

1. **donator (P0)** runs first among ready equals? Actually highest ready runs — but initially all start; priorities decide. `donator` records `0`, sleeps 30.
2. **spinner (P1)** records `1`, sleeps 10, then yields forever — a classic mid-priority CPU consumer.
3. **locker1 (P2)** records `2`, locks `mutex1`, sleeps 20 (still holding `mutex1`), later locks `mutex2`.
4. **locker0 (P3)** records `3`, needs `mutex0` then `mutex1`. If `donator` still holds / will hold `mutex0`, and `locker1` holds `mutex1`, classic inversion appears when a low-priority holder of a lock needed by high priority is postponed behind the spinner — **unless** the mutex implementation donates priority from waiter to holder.
5. After unlocks, `donator` records 6–7, fails a `rt_mutex_timedlock(&mutex2, 10)` on purpose (`assert(!…)`), then 9–10 and exits when `seq` hits `MAX_SEQ`.

The sleeps stagger lock acquisitions so the inversion window is reproducible. Priority numbers matter: P3 must outrank P2 and P1 for donation to lift the low holder above the spinner.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_MUTEX` / `rt_mutex_lock` / `unlock` | Nested critical sections that create inversion. |
| `rt_mutex_timedlock` | Negative test that `mutex2` stays held. |
| `rt_atomic_*` | Sequence counter without extra locking. |
| `rt_assert` | Enforce exact ordering (`sequence`). |
| `rt_task_sleep` / `rt_task_yield` | Timing and mid-priority busy loop. |
| `rt_exit` | Success when `seq` reaches `MAX_SEQ`. |

## Success / failure

- **Pass:** all `sequence` asserts hold; `rt_exit` at seq 10.
- **Fail:** `"sequence out of order"` assert, `"donator timedlock succeeded"`, or hang if donation broken and spinner runs forever while high priority waits.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/donate
```

## See also

- [Mutexes](../03-Kernel-Features/Mutexes.md) · [Scheduling](../03-Kernel-Features/Scheduling.md)
- [mutex](mutex.md) · [fair](fair.md)
- [Building and Examples](../07-Building-and-Examples.md)
