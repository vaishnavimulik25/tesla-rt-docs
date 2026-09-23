# `rt/event.h`

**Audience:** application.

## Types

- `struct rt_event`

## Macros

| Macro |
|-------|
| `RT_EVENT_INIT` |
| `RT_EVENT` |

## Functions

### `rt_event_clear`

```c
uint32_t rt_event_clear(struct rt_event *, uint32_t);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_get`

```c
uint32_t rt_event_get(const struct rt_event *);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_set`

```c
uint32_t rt_event_set(struct rt_event *, uint32_t);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_set_from_task`

```c
uint32_t rt_event_set_from_task(struct rt_event *, uint32_t);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_set_from_interrupt`

```c
uint32_t rt_event_set_from_interrupt(struct rt_event *, uint32_t);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_wait`

```c
uint32_t rt_event_wait(struct rt_event *, uint32_t);
```

- **Blocking behavior:** May block
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_trywait`

```c
uint32_t rt_event_trywait(struct rt_event *, uint32_t);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_timedwait`

```c
uint32_t rt_event_timedwait(struct rt_event *, uint32_t, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

### `rt_event_bits_match`

```c
bool rt_event_bits_match(uint32_t bits, uint32_t wait);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `set` ISR-safe. Blocking wait task-only. get/clear/trywait non-blocking.
- **Examples:** `examples/event.c`, `examples/cycle/event.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/event.h`
