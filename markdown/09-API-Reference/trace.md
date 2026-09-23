# `rt/trace.h`

**Audience:** advanced / internal hooks.

!!! warning "Internal / advanced"
    Intended for ports, tracing, or kernel internals. Application code should prefer task/sync/timer APIs.

## Types

- `struct rt_trace_event`

## Enumerations

- `enum rt_syscall`
- `enum rt_syscall_pendable`
- `enum rt_task_state`
- `enum rt_trace_event_state`
- `enum rt_trace_event_type`

## Macros

| Macro |
|-------|
| `RT_TRACE_ENABLE` |

## Functions

### `rt_trace_syscall_run`

```c
void rt_trace_syscall_run(enum rt_syscall syscall, uintptr_t arg0, uintptr_t arg1, uintptr_t arg2);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_syscall_run_pending`

```c
void rt_trace_syscall_run_pending(const struct rt_syscall_record *record);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_syscall_pend`

```c
void rt_trace_syscall_pend(const struct rt_syscall_record *record);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_first_task`

```c
void rt_trace_first_task(const struct rt_task *task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_active_task`

```c
void rt_trace_active_task(const struct rt_task *old_task, const struct rt_task *new_task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_task_state`

```c
void rt_trace_task_state(const struct rt_task *task, enum rt_task_state old_state, enum rt_task_state new_state);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_tick`

```c
void rt_trace_tick(unsigned long tick);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_interrupt_start`

```c
void rt_trace_interrupt_start(int32_t interrupt_number);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_interrupt_end`

```c
void rt_trace_interrupt_end(int32_t interrupt_number);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_sem_update`

```c
void rt_trace_sem_update(const struct rt_sem *sem, int old_value, int new_value);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_event_wait`

```c
void rt_trace_event_wait(const struct rt_event *event, uint32_t bits, uint32_t wait);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_event_set`

```c
void rt_trace_event_set(const struct rt_event *event, uint32_t bits, uint32_t set);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_event_clear`

```c
void rt_trace_event_clear(const struct rt_event *event, uint32_t bits, uint32_t clear);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_notify_or`

```c
void rt_trace_notify_or(const struct rt_notify *note, uint32_t old_value, uint32_t or_value, uint32_t new_value);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_notify_set`

```c
void rt_trace_notify_set(const struct rt_notify *note, uint32_t old_value, uint32_t new_value);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_notify_clear`

```c
void rt_trace_notify_clear(const struct rt_notify *note, uint32_t old_value, uint32_t clear_mask, uint32_t new_value);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_mutex_lock`

```c
void rt_trace_mutex_lock(const struct rt_mutex *mutex, uintptr_t holder);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_mutex_lock_fail`

```c
void rt_trace_mutex_lock_fail(const struct rt_mutex *mutex, uintptr_t failer, uintptr_t holder);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_mutex_unlock`

```c
void rt_trace_mutex_unlock(const struct rt_mutex *mutex, uintptr_t holder);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_timer_start`

```c
void rt_trace_timer_start(const struct rt_timer *timer, unsigned long period);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_timer_stop`

```c
void rt_trace_timer_stop(const struct rt_timer *timer);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_timer_change_period`

```c
void rt_trace_timer_change_period(const struct rt_timer *timer, unsigned long period);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_timer_expire`

```c
void rt_trace_timer_expire(const struct rt_timer *timer, unsigned long tick);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_queue_push`

```c
void rt_trace_queue_push(const struct rt_queue *queue, size_t enq);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_queue_push_skipped`

```c
void rt_trace_queue_push_skipped(const struct rt_queue *queue, size_t enq);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_queue_pop`

```c
void rt_trace_queue_pop(const struct rt_queue *queue, size_t deq);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_queue_pop_skip`

```c
void rt_trace_queue_pop_skip(const struct rt_queue *queue, size_t deq);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_queue_peek`

```c
void rt_trace_queue_peek(const struct rt_queue *queue, size_t deq);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_queue_peek_skip`

```c
void rt_trace_queue_peek_skip(const struct rt_queue *queue, size_t deq);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_end`

```c
void rt_trace_end(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_flush`

```c
void rt_trace_flush(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `__attribute__`

```c
__attribute__((weak)) bool rt_trace_event_write(const struct rt_trace_event *event);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_trace_daemon`

```c
void rt_trace_daemon(void);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/trace.h`
