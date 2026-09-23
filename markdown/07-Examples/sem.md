# Example: `sem.c`

Poster / waiter handshake on a counting semaphore with timed waits and an intentional timeout.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `poster` | 0 | `RT_STACK_MIN` | Sleeps 5 ticks, posts `n` times; later posts one more after 15 ticks. |
| `waiter` | 0 | `RT_STACK_MIN` | Timed-waits for each of `n` posts, then asserts a wait *times out*, then `rt_exit`. |

Both priority `0` (equal). Timing (sleeps / timeouts) drives the handshake rather than priority preemption.

## Shared state

```c
static const int n = 10;
static RT_SEM(sem, 0);   /* counting semaphore, initial count 0 */
```

## Control flow

1. `waiter` blocks in `rt_sem_timedwait(&sem, 10)` until a post arrives (or 10 ticks elapse).
2. `poster` sleeps 5 ticks between each of `n` posts — within the waiter’s 10-tick budget, so each of the first `n` waits succeeds.
3. After the loop, `poster` sleeps 15 ticks before the extra post.
4. `waiter` deliberately uses another 10-tick timed wait and asserts it **fails** (`!rt_sem_timedwait`) — proving timeout works when nothing is posted yet.
5. `waiter` calls `rt_exit()` (the late post from `poster` is irrelevant after exit).

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_SEM(sem, 0)` | Static counting semaphore, empty at start. |
| `rt_sem_post` | Increment count / wake a waiter. |
| `rt_sem_timedwait` | Wait up to N ticks; returns bool success. |
| `rt_task_sleep` | Space posts and create the intentional timeout window. |
| `rt_assert` | Fail the example if timed wait succeeds/fails unexpectedly. |
| `rt_exit` | Pass when the timeout negative case is confirmed. |

## Success / failure

- **Pass:** all `n` waits succeed; the following wait times out; `rt_exit`.
- **Fail:** `rt_assert` on unexpected timeout / non-timeout, or hang if posts never arrive.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/sem
```

## See also

- [Semaphores](../03-Kernel-Features/Semaphores.md)
- [notify](notify.md) · [cycle-sem](cycle-sem.md) · [water-sem](water-sem.md)
- [Building and Examples](../07-Building-and-Examples.md)
