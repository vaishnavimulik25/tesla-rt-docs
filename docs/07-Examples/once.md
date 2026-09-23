# Example: `once.c`

Two tasks hammer `rt_once_call`: one resets the once flag after each call so the init function runs exactly `ITERATIONS` times under contention.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `oncer` | 0 | `2 * RT_STACK_MIN` | Loop: `rt_once_call(&once, fn)` then `rt_sem_wait`. |
| `oncer_reset` | 0 | `2 * RT_STACK_MIN` | Loop: `rt_once_call`, store `once.done = 0`, `rt_sem_post`; finally assert `x == ITERATIONS` and `rt_exit`. |

`ITERATIONS = 10000`.

## Shared state

```c
static RT_ONCE(once);
static RT_SEM(sem, 0);
static rt_atomic_ulong x = 0;

static void fn(void)
{
    rt_atomic_fetch_add(&x, 1, RT_ATOMIC_RELAXED);
}
```

## Control flow

1. Both tasks call `rt_once_call` every iteration. Without reset, `fn` would run only once for the process lifetime.
2. `oncer_reset` deliberately clears `once.done` with a release store after each call, re-arming the once object.
3. The semaphore pairs the two loops: `oncer` waits for a post after each call; `oncer_reset` posts after reset — keeping them in lockstep so contention and reset ordering are exercised together.
4. After 10000 cycles, `x` must equal `ITERATIONS` (one increment of `fn` per re-armed call cycle from the resetter’s perspective / coordinated calls).
5. Assert and `rt_exit`.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_ONCE` / `rt_once_call` | One-shot init call (re-armed manually here for stress). |
| `rt_atomic_*` | Count `fn` invocations; clear `once.done`. |
| `RT_SEM` / `wait` / `post` | Pair the two tasks each iteration. |

## Success / failure

- **Pass:** `x == 10000`, `rt_exit`.
- **Fail:** `"x was not incremented enough"` or hang on the semaphore.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/once
```

## See also

- [Other Primitives](../03-Kernel-Features/Other-Primitives.md) (once)
- [API: once](../09-API-Reference/once.md)
- [Building and Examples](../07-Building-and-Examples.md)
