# `rt/mutex.h`

**Audience:** application.

## Types

- `struct rt_mutex`

## Macros

| Macro |
|-------|
| `RT_MUTEX_GUARD_ADOPT` |
| `RT_MUTEX_GUARD` |
| `RT_MUTEX_UNLOCKED` |
| `RT_MUTEX_WAITED_MASK` |
| `RT_MUTEX_HOLDER_MASK` |
| `RT_MUTEX_HOLDER_INTERRUPT` |
| `RT_MUTEX_LEVEL_NONRECURSIVE` |
| `RT_MUTEX_INIT_COMMON` |
| `RT_MUTEX_INIT` |
| `RT_MUTEX_INIT_RECURSIVE` |
| `RT_MUTEX` |
| `RT_MUTEX_RECURSIVE` |

## Functions

### `rt_mutex_lock`

```c
void rt_mutex_lock(struct rt_mutex *mutex);
```

- **Blocking behavior:** May block
- **ISR:** Blocking lock/timedlock (ticks>0) forbidden in ISR. `trylock` / `timedlock(0)` and unlock (if ISR holds) allowed.
- **Examples:** `examples/mutex.c`, `examples/recursive.c`, `examples/donate.c`, `examples/fair.c`, `examples/cond.c`

### `rt_mutex_unlock`

```c
void rt_mutex_unlock(struct rt_mutex *mutex);
```

- **Blocking behavior:** May block
- **ISR:** Blocking lock/timedlock (ticks>0) forbidden in ISR. `trylock` / `timedlock(0)` and unlock (if ISR holds) allowed.
- **Examples:** `examples/mutex.c`, `examples/recursive.c`, `examples/donate.c`, `examples/fair.c`, `examples/cond.c`

### `rt_mutex_trylock`

```c
bool rt_mutex_trylock(struct rt_mutex *mutex);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** Blocking lock/timedlock (ticks>0) forbidden in ISR. `trylock` / `timedlock(0)` and unlock (if ISR holds) allowed.
- **Examples:** `examples/mutex.c`, `examples/recursive.c`, `examples/donate.c`, `examples/fair.c`, `examples/cond.c`

### `rt_mutex_timedlock`

```c
bool rt_mutex_timedlock(struct rt_mutex *mutex, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** Blocking lock/timedlock (ticks>0) forbidden in ISR. `trylock` / `timedlock(0)` and unlock (if ISR holds) allowed.
- **Examples:** `examples/mutex.c`, `examples/recursive.c`, `examples/donate.c`, `examples/fair.c`, `examples/cond.c`

### `rt_mutex_unlock_guard`

```c
static inline void rt_mutex_unlock_guard(struct rt_mutex *const *pguard) { rt_mutex_unlock(*pguard);
```

- **Blocking behavior:** May block
- **ISR:** Blocking lock/timedlock (ticks>0) forbidden in ISR. `trylock` / `timedlock(0)` and unlock (if ISR holds) allowed.
- **Examples:** `examples/mutex.c`, `examples/recursive.c`, `examples/donate.c`, `examples/fair.c`, `examples/cond.c`

### `rt_mutex_lock`

```c
rt_mutex_lock(&(mutex));
```

- **Blocking behavior:** May block
- **ISR:** Blocking lock/timedlock (ticks>0) forbidden in ISR. `trylock` / `timedlock(0)` and unlock (if ISR holds) allowed.
- **Examples:** `examples/mutex.c`, `examples/recursive.c`, `examples/donate.c`, `examples/fair.c`, `examples/cond.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/mutex.h`
