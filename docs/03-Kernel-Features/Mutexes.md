# Mutexes

Header: [`include/rt/mutex.h`](https://git.rtng.org/rt/rt/src/branch/master/include/rt/mutex.h). Implementation: `src/mutex.c`, with priority donation in `src/rt.c`.

## Purpose and when to use

A mutex (`struct rt_mutex`) provides **mutual exclusion** for a critical section. Use a mutex when:

- Multiple tasks share data or hardware and must not race.
- You need **priority donation** (inheritance-style) so a low-priority holder is boosted by higher-priority waiters (see below).
- You want optional **recursive** locking of the same mutex by the same task.

Prefer a [semaphore](Semaphores.md) for pure signaling / resource counting without an owner, or a [queue](Queues.md) / [event](Events.md) for message-style coordination.

## Data type and static initialization

```c
struct rt_mutex {
    rt_atomic_uintptr_t holder;
    struct rt_list wait_list;
    struct rt_list list;   /* on holder's owned-mutex list (donation) */
    int level;             /* recursion depth; -1 = non-recursive */
};
```

```c
RT_MUTEX(name);              /* non-recursive: level = RT_MUTEX_LEVEL_NONRECURSIVE (-1) */
RT_MUTEX_RECURSIVE(name);    /* recursive: level starts at 0 */

/* Or initialize an existing object: */
struct rt_mutex m = RT_MUTEX_INIT(m);
struct rt_mutex r = RT_MUTEX_INIT_RECURSIVE(r);
```

Holder encoding (internal, but useful when reading traces / debugging):

| Macro | Meaning |
|-------|---------|
| `RT_MUTEX_UNLOCKED` | Free |
| `RT_MUTEX_WAITED_MASK` | Low bit: at least one waiter |
| `RT_MUTEX_HOLDER_MASK` | Holder pointer bits |
| `RT_MUTEX_HOLDER_INTERRUPT` | Special holder value when locked from an interrupt via try/timed(0) |
| `RT_MUTEX_LEVEL_NONRECURSIVE` | `(-1)` |

## Public API

```c
void rt_mutex_lock(struct rt_mutex *mutex);
void rt_mutex_unlock(struct rt_mutex *mutex);
bool rt_mutex_trylock(struct rt_mutex *mutex);
bool rt_mutex_timedlock(struct rt_mutex *mutex, unsigned long ticks);
```

| Function | Blocks? | Returns | Semantics |
|----------|---------|---------|-----------|
| `rt_mutex_lock` | Yes, until acquired | void | Asserts **not** in interrupt. Fast-path CAS; else syscall. |
| `rt_mutex_trylock` | Never | `true` if locked | Allowed from **task or interrupt**. Interrupt holders use `RT_MUTEX_HOLDER_INTERRUPT`. |
| `rt_mutex_timedlock` | Up to `ticks` | `true` if locked | From interrupt, only `ticks == 0` is allowed (same as try). |
| `rt_mutex_unlock` | May schedule | void | Must be called by the current holder (task or interrupt). Decrements recursion first; last unlock wakes waiters / clears holder. |

### Scope guards (GCC/Clang `__attribute__((cleanup))`)

```c
RT_MUTEX_GUARD(guard, mutex);        /* lock now; unlock when guard leaves scope */
RT_MUTEX_GUARD_ADOPT(guard, mutex);  /* adopt an already-locked mutex */
```

`rt_mutex_unlock_guard` is the cleanup helper; application code normally uses the macros only. Toolchains without `cleanup` (e.g. some TI C28 builds) omit mutex examples that rely on guards.

## Priority donation

**Present and automatic** for task-held mutexes.

When a task blocks on a mutex, `mutex_donate` / `task_donate` in `src/rt.c`:

1. Recalculate the holder’s effective `task->priority` as the **minimum** of `base_priority` and the best (highest) waiter priorities across mutexes on the holder’s `mutex_list` that have waiters (lower numeric value = higher priority).
2. Donation can **chain**: if the holder is itself waiting on another mutex, that upstream holder is re-evaluated.
3. On unlock or waiter timeout/removal, priorities are recalculated again.

Interrupts that hold a mutex via `trylock` are **not** priority-boosted (there is no task to donate to); the implementation comments that an interrupt should not hold a mutex long enough for waiters to accumulate.

See [examples/donate.c](../07-Examples/donate.md) for a nested inversion scenario with sequence asserts.

## ISR-safety rules

| API | From task | From ISR |
|-----|-----------|----------|
| `rt_mutex_lock` | Yes | **No** (assert) |
| `rt_mutex_timedlock` (`ticks > 0`) | Yes | **No** (assert) |
| `rt_mutex_timedlock` (`ticks == 0`) | Yes | Yes (try) |
| `rt_mutex_trylock` | Yes | Yes |
| `rt_mutex_unlock` | Yes (if holder) | Yes (if ISR was the holder via try) |

Do **not** recursively lock from an interrupt (assert). Prefer keeping ISR critical sections free of mutexes; use try-lock only for rare shared data, or post a [semaphore](Semaphores.md) / [event](Events.md) / [pend function](../09-API-Reference/pend_function.md) instead.

## Recursive locking

Use `RT_MUTEX_RECURSIVE`. Each successful lock by the same holder increments `level` (0 = locked once). Each unlock decrements until the last release. Non-recursive mutexes assert on re-lock by the same task.

## Common pitfalls

1. **Locking a non-recursive mutex twice** in the same task → assert.
2. **Unlocking without holding** → assert.
3. **Blocking lock from ISR** → assert; use `trylock` or another primitive.
4. **Assuming FIFO fairness among equal priorities** — wait lists are **priority-ordered**; same-priority ordering depends on insertion. See `examples/fair.c`.
5. **Long critical sections** while holding a mutex increase donation duration and can hurt lower-priority throughput.
6. **Guards on C28** — examples that use `RT_MUTEX_GUARD` are skipped when the compiler lacks `cleanup`.

## Related examples

| Example | Demonstrates |
|---------|----------------|
| `examples/mutex.c` | lock / trylock / timedlock + guards |
| `examples/recursive.c` | recursive mutex |
| `examples/donate.c` | priority donation / inversion avoidance |
| `examples/fair.c` | contention among equal-priority tasks |
| `examples/cond.c` | mutex + condition variable |
| `examples/cycle/mutex.c` | cycle-count micro-benchmark of contended lock |
