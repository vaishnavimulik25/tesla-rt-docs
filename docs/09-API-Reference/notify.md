# `rt/notify.h`

**Audience:** application.

## Types

- `struct rt_notify`

## Macros

| Macro |
|-------|
| `RT_NOTIFY_INIT` |
| `RT_NOTIFY` |

## Functions

### `rt_notify_post`

```c
void rt_notify_post(struct rt_notify *note);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_or`

```c
void rt_notify_or(struct rt_notify *note, uint32_t value);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_set`

```c
void rt_notify_set(struct rt_notify *note, uint32_t value);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_wait`

```c
uint32_t rt_notify_wait(struct rt_notify *note);
```

- **Blocking behavior:** May block
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_wait_clear`

```c
uint32_t rt_notify_wait_clear(struct rt_notify *note, uint32_t clear);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_trywait`

```c
bool rt_notify_trywait(struct rt_notify *note, uint32_t *value);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_trywait_clear`

```c
bool rt_notify_trywait_clear(struct rt_notify *note, uint32_t *value, uint32_t clear);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_timedwait`

```c
bool rt_notify_timedwait(struct rt_notify *note, uint32_t *value, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

### `rt_notify_timedwait_clear`

```c
bool rt_notify_timedwait_clear(struct rt_notify *note, uint32_t *value, uint32_t clear, unsigned long ticks);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** post/or/set ISR-capable via sem. Waits task-only.
- **Examples:** `examples/notify.c`, `examples/cycle/notify.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/notify.h`
