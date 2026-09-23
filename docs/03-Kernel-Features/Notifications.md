# Notifications

Header: `include/rt/notify.h`. Implementation: `src/notify.c`.

## Purpose and when to use

A notification object pairs a **binary semaphore** with an atomic `uint32_t` value. It is a lightweight alternative to a full event group when a single waiter (or simple post/wait pair) needs a small payload.

Use notifications when:

- One task waits for a wake with an optional 32-bit value (`post` / `or` / `set`).
- You want try/timed wait with optional clear-on-read.

## Data type and static initialization

```c
struct rt_notify {
    rt_atomic_uint32_t value;
    struct rt_sem sem;     /* binary */
};

RT_NOTIFY(name, initial_value);
struct rt_notify n = RT_NOTIFY_INIT(n, 0);
```

## Public API

```c
void rt_notify_post(struct rt_notify *note);           /* wake, value unchanged */
void rt_notify_or(struct rt_notify *note, uint32_t value);  /* value |= */
void rt_notify_set(struct rt_notify *note, uint32_t value); /* value = */

uint32_t rt_notify_wait(struct rt_notify *note);
uint32_t rt_notify_wait_clear(struct rt_notify *note, uint32_t clear);

bool rt_notify_trywait(struct rt_notify *note, uint32_t *value);
bool rt_notify_trywait_clear(struct rt_notify *note, uint32_t *value,
                             uint32_t clear);

bool rt_notify_timedwait(struct rt_notify *note, uint32_t *value,
                         unsigned long ticks);
bool rt_notify_timedwait_clear(struct rt_notify *note, uint32_t *value,
                               uint32_t clear, unsigned long ticks);
```

| Function | Blocks? | Value behavior |
|----------|---------|----------------|
| `post` | No | Leaves `value` unchanged; posts the binary sem |
| `or` | No | `value \|=` then post |
| `set` | No | `value =` then post |
| `wait` | Yes | Returns current value after taking the sem |
| `wait_clear` | Yes | Returns value **before** `value &= ~clear` |
| `trywait` | No | `false` if no pending notification |
| `trywait_clear` | No | Same + clear mask |
| `timedwait` / `_clear` | Timed | `false` on timeout |

## Priority donation

None (semaphore wake ordering only).

## ISR-safety rules

Post/`or`/`set` go through `rt_sem_post` and are ISR-capable. Blocking waits are task-only. Use try/timed(0) patterns if you must poll from unusual contexts.

## Common pitfalls

1. Multiple waiters on one `rt_notify` — binary sem wakes **one**; others need another post.
2. Confusing `post` (no value change) with `set`.
3. Forgetting that `wait_clear` returns the pre-clear value.
4. Treating notifications as a multi-bit event group — use [events](Events.md) for rich bit waits.

## Related examples

| Example | Demonstrates |
|---------|----------------|
| `examples/notify.c` | post/wait between tasks |
| `examples/cycle/notify.c` | post→wait cycle latency |
