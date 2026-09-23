# Semaphores

Header: `include/rt/sem.h`. Implementation: `src/sem.c`.

## Purpose and when to use

A counting semaphore signals availability of **N** units of a resource or synchronizes producer/consumer tasks without transferring a data payload.

Use a semaphore when:

- An ISR or task must wake one or more waiters (`post` / `post_n`).
- You need a **binary** semaphore (max value 1) for simple sync.
- You want a counting limit (`RT_SEM_MAX`).

Prefer a [mutex](Mutexes.md) when you need ownership and priority donation; prefer a [queue](Queues.md) when you must pass values; prefer a [notification](Notifications.md) for a lightweight task-targeted wake with optional bit payload.

## Data type and static initialization

```c
struct rt_sem {
    rt_atomic_int value;
    int max_value;
    struct rt_list wait_list;
    size_t num_waiters;
    struct rt_syscall_record post_record; /* for ISR-safe pendable post */
};
```

```c
RT_SEM(name, count);              /* initial count; max = INT_MAX */
RT_SEM_MAX(name, count, max);     /* capped counting semaphore */
RT_SEM_BINARY(name);              /* initial 0, max 1 */

struct rt_sem s = RT_SEM_INIT(s, 0);
struct rt_sem b = RT_SEM_INIT_BINARY(b);
struct rt_sem m = RT_SEM_INIT_MAX(m, 0, 4);
```

## Public API

```c
void rt_sem_post(struct rt_sem *sem);
void rt_sem_post_from_task(struct rt_sem *sem);
void rt_sem_post_from_interrupt(struct rt_sem *sem);

void rt_sem_post_n(struct rt_sem *sem, int n);
void rt_sem_post_n_from_task(struct rt_sem *sem, int n);
void rt_sem_post_n_from_interrupt(struct rt_sem *sem, int n);

void rt_sem_wait(struct rt_sem *sem);
bool rt_sem_trywait(struct rt_sem *sem);
bool rt_sem_timedwait(struct rt_sem *sem, unsigned long ticks);

void rt_sem_add_n(struct rt_sem *sem, int n);
```

| Function | Blocks? | ISR | Notes |
|----------|---------|-----|-------|
| `rt_sem_post` / `post_n` | No | Yes (auto) | Detects ISR via `rt_interrupt_is_active()`; uses fast atomic path or pendable syscall record. |
| `rt_sem_post_from_task` / `_n` | No | Prefer not | Always uses task/syscall path when waiters exist. |
| `rt_sem_post_from_interrupt` / `_n` | No | Yes | Always uses the semaphore’s `post_record` when the fast path cannot complete. |
| `rt_sem_wait` | Yes | **No** | Decrements or blocks until posted. |
| `rt_sem_trywait` | No | Yes (non-blocking) | Returns `false` if value would go below 1. |
| `rt_sem_timedwait` | Up to `ticks` | No (blocking) | Returns `false` on timeout. |
| `rt_sem_add_n` | No | Yes | Adds up to `max_value` **without waking waiters** on the waiter path the same way as post when value was negative—used internally when a post record is already pending; treat as advanced. Prefer `post_n` from application code. |

Values are saturating at `max_value` on post. When waiters exist (`value < 0` on the slow path), posting goes through a syscall so wakes respect priority order.

## Priority donation

Semaphores do **not** implement priority inheritance to a “holder.” Wait lists are priority-sorted for wake order. Mutexes are the primitive that donates priority to an owner.

## ISR-safety rules

- **Post from ISR:** supported. Prefer `rt_sem_post` / `rt_sem_post_n` (auto) or the `_from_interrupt` variants.
- **Wait / timedwait from ISR:** not appropriate (blocking syscalls).
- **Trywait from ISR:** OK if you only need a non-blocking decrement.
- Only one pendable `post_record` exists per semaphore; if a post is already pending, additional ISR posts fall back to `rt_sem_add_n` so counts are not lost (see comments in `src/sem.c`).

## Common pitfalls

1. Using a semaphore as a mutex **without** ownership/donation semantics.
2. Forgetting `max_value` when you need a binary semaphore — use `RT_SEM_BINARY`.
3. Calling blocking wait from an ISR.
4. Assuming posts always immediately schedule the highest waiter — pendable posts complete in the pending syscall handler before the next task runs, but nested interrupt races use the add-n fallback.
5. Overflow: posts saturate at `max_value`; excess is discarded by saturation logic.

## Related examples

| Example | Demonstrates |
|---------|----------------|
| `examples/sem.c` | post / wait between tasks |
| `examples/water/sem.c` | classic H₂O bonding with semaphores |
| `examples/cycle/sem.c` | post→wait latency in CPU cycles |
| `examples/fair.c` | exit rendezvous via trywait on a semaphore |
