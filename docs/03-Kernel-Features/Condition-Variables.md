# Condition Variables

Header: `include/rt/cond.h`. Implementation: `src/cond.c` (built on a binary semaphore).

## Purpose and when to use

A condition variable (`struct rt_cond`) lets a task **wait for a predicate** while temporarily releasing a mutex, then re-acquire the mutex when signaled. Use it when:

- Shared state is protected by a [mutex](Mutexes.md), and waiters must sleep until another task changes that state.
- You need Mesa-style “signal / broadcast” wakeups (always re-check the predicate in a loop).

`rt_cond` is a thin wrapper: the object contains a **binary semaphore**. Waiting drops the mutex, waits on that semaphore, then locks the mutex again.

## Data type and static initialization

```c
struct rt_cond {
    struct rt_sem sem;   /* binary semaphore */
};

RT_COND(name);
struct rt_cond c = RT_COND_INIT(c);
```

## Public API

```c
void rt_cond_signal(struct rt_cond *cond);
void rt_cond_broadcast(struct rt_cond *cond);
void rt_cond_wait(struct rt_cond *cond, struct rt_mutex *mutex);
bool rt_cond_timedwait(struct rt_cond *cond, struct rt_mutex *mutex,
                       unsigned long ticks);
```

| Function | Blocks? | Holds mutex on entry? | Semantics |
|----------|---------|----------------------|-----------|
| `rt_cond_wait` | Yes | **Yes — must hold `mutex`** | Unlock `mutex`, wait for signal/broadcast, re-lock `mutex` before return. |
| `rt_cond_timedwait` | Up to `ticks` | Yes | Same as wait; returns `false` if the wait timed out (mutex is still re-locked). |
| `rt_cond_signal` | No | Optional | Wakes **one** waiter (posts the binary sem). |
| `rt_cond_broadcast` | No | Optional | Wakes **all** waiters (`post_n` with waiter count). |

### Correct wait pattern

```c
RT_MUTEX(lock);
RT_COND(cv);
bool ready = false;

void consumer(void)
{
    rt_mutex_lock(&lock);
    while (!ready) {
        rt_cond_wait(&cv, &lock);   /* re-check predicate */
    }
    /* use shared state */
    rt_mutex_unlock(&lock);
}

void producer(void)
{
    rt_mutex_lock(&lock);
    ready = true;
    rt_cond_signal(&cv);            /* or broadcast */
    rt_mutex_unlock(&lock);
}
```

With a guard:

```c
RT_MUTEX_GUARD(g, lock);
while (!ready) {
    rt_cond_wait(&cv, g);  /* pass the mutex pointer held by the guard */
}
```

(`RT_MUTEX_GUARD` defines `g` as `struct rt_mutex * const`, so pass `g` or `&lock` consistently with your macro expansion — the public wait API takes `struct rt_mutex *`.)

## Priority donation

While blocked in `rt_cond_wait`, the task is waiting on the **condition’s semaphore**, not holding the mutex. The mutex is released for the duration of the wait, so donation applies only while the mutex is actually held (before wait / after wake). Signal does not donate priority by itself.

## ISR-safety rules

| API | Task | ISR |
|-----|------|-----|
| `rt_cond_wait` / `timedwait` | Yes | **No** (releases mutex + blocks) |
| `rt_cond_signal` / `broadcast` | Yes | Possible via underlying `rt_sem_post*` paths, but **typical app code signals from tasks** holding the same protocol as the waiters. Prefer signaling from task context after updating the predicate under the mutex. |

## Common pitfalls

1. **Forgetting the `while (!predicate)` loop** — signals can be consumed by another waiter; spurious-style wakeups are possible with Mesa semantics.
2. **Waiting without holding the mutex** — undefined / assert-prone protocol break.
3. **Using cond without a mutex** — not supported; the API requires a mutex.
4. **Broadcast storms** — every waiter re-locks the mutex serially; that is expected.
5. **Toolchain without `cleanup`** — examples that use `RT_MUTEX_GUARD` are omitted on some C28 builds (`examples/SConscript`).

## Related examples

| Example | Demonstrates |
|---------|----------------|
| `examples/cond.c` | signaler + waiters + mutex guard |
| `examples/water/cond.c` | H₂O bonding with two condition variables |
