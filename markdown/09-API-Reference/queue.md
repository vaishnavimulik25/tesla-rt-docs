# `rt/queue.h`

**Audience:** application.

## Types

- `struct rt_queue`

## Macros

| Macro |
|-------|
| `RT_QUEUE_STATE_BITS` |
| `RT_QUEUE_GEN_BITS` |
| `RT_QUEUE_SIZE_BITS` |
| `RT_QUEUE_INDEX_BITS` |
| `RT_QUEUE_MAX_SIZE` |
| `RT_QUEUE_INIT` |
| `RT_QUEUE_COMMON` |
| `RT_QUEUE` |
| `RT_QUEUE_STATIC` |
| `RT_QUEUE_QGEN_INCREMENT` |
| `RT_QUEUE_QINDEX_MASK` |
| `RT_QUEUE_QGEN_MASK` |
| `RT_QUEUE_QSGEN_SHIFT` |

## Functions

### `rt_queue_push`

```c
void rt_queue_push(struct rt_queue *queue, const void *elem);
```

- **Blocking behavior:** May block
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_pop`

```c
void rt_queue_pop(struct rt_queue *queue, void *elem);
```

- **Blocking behavior:** May block
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_peek`

```c
void rt_queue_peek(struct rt_queue *queue, void *elem);
```

- **Blocking behavior:** May block
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_trypush`

```c
bool rt_queue_trypush(struct rt_queue *queue, const void *elem);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_trypop`

```c
bool rt_queue_trypop(struct rt_queue *queue, void *elem);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_trypeek`

```c
bool rt_queue_trypeek(struct rt_queue *queue, void *elem);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_timedpush`

```c
bool rt_queue_timedpush(struct rt_queue *queue, const void *elem, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_timedpop`

```c
bool rt_queue_timedpop(struct rt_queue *queue, void *elem, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_timedpeek`

```c
bool rt_queue_timedpeek(struct rt_queue *queue, void *elem, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_static_assert`

```c
rt_static_assert((num) <= RT_QUEUE_MAX_SIZE, "queue is too large");
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

### `rt_queue_qsgen`

```c
static inline unsigned char rt_queue_qsgen(size_t q) { return (unsigned char)(rt_queue_qgen(q) >> RT_QUEUE_QSGEN_SHIFT);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Blocking push/pop/peek task-only. Prefer trypush from ISR.
- **Examples:** `examples/queue.c`, `examples/cycle/queue.c`, `examples/timer.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/queue.h`
