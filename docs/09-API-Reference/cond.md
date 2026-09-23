# `rt/cond.h`

**Audience:** application.

## Types

- `struct rt_cond`

## Macros

| Macro |
|-------|
| `RT_COND_INIT` |
| `RT_COND` |

## Functions

### `rt_cond_signal`

```c
void rt_cond_signal(struct rt_cond *cond);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Wait APIs are task-only. Signal/broadcast via sem post paths.
- **Examples:** `examples/cond.c`, `examples/water/cond.c`

### `rt_cond_broadcast`

```c
void rt_cond_broadcast(struct rt_cond *cond);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Wait APIs are task-only. Signal/broadcast via sem post paths.
- **Examples:** `examples/cond.c`, `examples/water/cond.c`

### `rt_cond_wait`

```c
void rt_cond_wait(struct rt_cond *cond, struct rt_mutex *mutex);
```

- **Blocking behavior:** May block
- **ISR:** Wait APIs are task-only. Signal/broadcast via sem post paths.
- **Examples:** `examples/cond.c`, `examples/water/cond.c`

### `rt_cond_timedwait`

```c
bool rt_cond_timedwait(struct rt_cond *cond, struct rt_mutex *mutex, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** Wait APIs are task-only. Signal/broadcast via sem post paths.
- **Examples:** `examples/cond.c`, `examples/water/cond.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/cond.h`
