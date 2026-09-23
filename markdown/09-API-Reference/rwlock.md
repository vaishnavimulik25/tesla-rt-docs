# `rt/rwlock.h`

**Audience:** application.

## Types

- `struct rt_rwlock`

## Macros

| Macro |
|-------|
| `RT_RWLOCK_READ_GUARD_ADOPT` |
| `RT_RWLOCK_READ_GUARD` |
| `RT_RWLOCK_WRITE_GUARD_ADOPT` |
| `RT_RWLOCK_WRITE_GUARD` |
| `RT_RWLOCK_INIT` |
| `RT_RWLOCK` |

## Functions

### `rt_rwlock_rdlock`

```c
void rt_rwlock_rdlock(struct rt_rwlock *lock);
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_tryrdlock`

```c
bool rt_rwlock_tryrdlock(struct rt_rwlock *lock);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_timedrdlock`

```c
bool rt_rwlock_timedrdlock(struct rt_rwlock *lock, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_rdunlock`

```c
void rt_rwlock_rdunlock(struct rt_rwlock *lock);
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_rdunlock_guard`

```c
static inline void rt_rwlock_rdunlock_guard(struct rt_rwlock *const *pguard) { rt_rwlock_rdunlock(*pguard);
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_rdlock`

```c
rt_rwlock_rdlock(&(lock));
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_wrlock`

```c
void rt_rwlock_wrlock(struct rt_rwlock *lock);
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_trywrlock`

```c
bool rt_rwlock_trywrlock(struct rt_rwlock *lock);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_timedwrlock`

```c
bool rt_rwlock_timedwrlock(struct rt_rwlock *lock, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_wrunlock`

```c
void rt_rwlock_wrunlock(struct rt_rwlock *lock);
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_wrunlock_guard`

```c
static inline void rt_rwlock_wrunlock_guard(struct rt_rwlock *const *pguard) { rt_rwlock_wrunlock(*pguard);
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_wrlock`

```c
rt_rwlock_wrlock(&(lock));
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

### `rt_rwlock_unlock`

```c
void rt_rwlock_unlock(struct rt_rwlock *lock);
```

- **Blocking behavior:** May block
- **ISR:** Blocking locks task-only.
- **Examples:** `examples/rwlock.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/rwlock.h`
