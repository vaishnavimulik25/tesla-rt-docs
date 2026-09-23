# Example: `cond.c`

Condition-variable handshake: one signaler sets `flag` and signals; two equal-priority incrementers wait until `flag` is true, clear it, and count to `ITERATIONS`.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `signaler` | 2 | `RT_STACK_MIN` | Forever: lock, `flag = true`, `rt_cond_signal`. |
| `incrementer` ×2 | 1 | `RT_STACK_MIN` | Wait for `flag`, `++x`, clear `flag`; after 1000 iters exit path. |
| `timeout` | 0 | `RT_STACK_MIN` | Sleep 1000; `rt_panic("timed out")`. |

Only **one** of the two incrementers actually drives completion: both share one loop body but each has its own stack/task; looking at the source, each incrementer independently runs `ITERATIONS` waits under one long mutex hold:

```c
RT_MUTEX_GUARD(guard, mutex);
for (int i = 0; i < ITERATIONS; ++i)
{
    while (!flag)
        rt_cond_wait(&cond, guard);
    ++x;
    flag = false;
}
rt_exit();
```

Whichever incrementer finishes first calls `rt_exit()` (both can call it after their loops). The higher-priority `signaler` keeps offering signals so waiters are not stuck.

## Shared state

```c
static RT_MUTEX(mutex);
static RT_COND(cond);
static unsigned x = 0;
static bool flag = false;
#define ITERATIONS 1000
```

## Control flow

1. Incrementer(s) hold `mutex`, wait in `while (!flag) rt_cond_wait` — wait atomically releases the mutex and re-acquires on wake (predicate loop avoids lost-wakeup / spurious wake bugs).
2. Signaler (P2) preempts / runs, sets `flag`, signals.
3. Incrementer wakes, sees `flag`, increments `x`, clears `flag`, loops.
4. After 1000 successful handshakes, incrementer `rt_exit`s.
5. If signaling/waiting breaks, timeout panics.

Priority: signaler > incrementers so a ready signaler can run even when incrementers are runnable after unlock; timeout is lowest as a watchdog only.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_MUTEX` / `RT_MUTEX_GUARD` | Protect `flag` / `x`. |
| `RT_COND` | Condition variable tied to that mutex. |
| `rt_cond_wait` | Sleep until signal; re-check predicate. |
| `rt_cond_signal` | Wake one waiter. |
| `rt_panic` | Watchdog. |

## Success / failure

- **Pass:** incrementer completes iterations and `rt_exit`.
- **Fail:** timeout panic, or hang in `cond_wait`.

> Skipped for `cl2000`.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/cond
```

## See also

- [Condition Variables](../03-Kernel-Features/Condition-Variables.md)
- [mutex](mutex.md) · [water-cond](water-cond.md)
- [Building and Examples](../07-Building-and-Examples.md)
