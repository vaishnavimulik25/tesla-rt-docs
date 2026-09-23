# `rt/syscall.h`

**Audience:** internal / port.

!!! warning "Internal / advanced"
    Intended for ports, tracing, or kernel internals. Application code should prefer task/sync/timer APIs.

## Types

- `struct rt_syscall_args_sem_post`
- `struct rt_syscall_args_event_set`
- `struct rt_syscall_args_function`
- `struct rt_syscall_record`

## Enumerations

- `enum rt_syscall`
- `enum rt_syscall_pendable`

## Macros

| Macro |
|-------|
| `RT_SYSCALL_NOIPA` |
| `RT_SYSCALL_NOIPA` |

## Functions

### `rt_syscall_0`

```c
RT_SYSCALL_NOIPA void rt_syscall_0(enum rt_syscall syscall);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_1`

```c
RT_SYSCALL_NOIPA void rt_syscall_1(enum rt_syscall syscall, uintptr_t arg0);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_2`

```c
RT_SYSCALL_NOIPA void rt_syscall_2(enum rt_syscall syscall, uintptr_t arg0, uintptr_t arg1);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_3`

```c
RT_SYSCALL_NOIPA void rt_syscall_3(enum rt_syscall syscall, uintptr_t arg0, uintptr_t arg1, uintptr_t arg2);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_sleep`

```c
static inline void rt_syscall_sleep(unsigned long ticks) { rt_syscall_1(RT_SYSCALL_SLEEP, ticks);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_sleep_periodic`

```c
static inline void rt_syscall_sleep_periodic(unsigned long last_wake_tick, unsigned long period) { rt_syscall_2(RT_SYSCALL_SLEEP_PERIODIC, last_wake_tick, period);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_sem_wait`

```c
static inline void rt_syscall_sem_wait(struct rt_sem *sem) { rt_syscall_1(RT_SYSCALL_SEM_WAIT, (uintptr_t)sem);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_3`

```c
rt_syscall_3(RT_SYSCALL_SEM_TIMEDWAIT, (uintptr_t)sem, ticks, (uintptr_t)&success);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_sem_post`

```c
static inline void rt_syscall_sem_post(struct rt_sem *sem, int n) { rt_syscall_2(RT_SYSCALL_SEM_POST, (uintptr_t)sem, (uintptr_t)n);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_mutex_lock`

```c
static inline void rt_syscall_mutex_lock(struct rt_mutex *mutex) { rt_syscall_1(RT_SYSCALL_MUTEX_LOCK, (uintptr_t)mutex);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_3`

```c
rt_syscall_3(RT_SYSCALL_MUTEX_TIMEDLOCK, (uintptr_t)mutex, ticks, (uintptr_t)&success);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_mutex_unlock`

```c
static inline void rt_syscall_mutex_unlock(struct rt_mutex *mutex) { rt_syscall_1(RT_SYSCALL_MUTEX_UNLOCK, (uintptr_t)mutex);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_event_wait`

```c
static inline uint32_t rt_syscall_event_wait(struct rt_event *event, uint32_t bits) { rt_syscall_2(RT_SYSCALL_EVENT_WAIT, (uintptr_t)event, (uintptr_t)&bits);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_event_timedwait`

```c
static inline uint32_t rt_syscall_event_timedwait(struct rt_event *event, uint32_t bits, unsigned long ticks) { rt_syscall_3(RT_SYSCALL_EVENT_TIMEDWAIT, (uintptr_t)event, (uintptr_t)&bits, ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_event_set`

```c
static inline void rt_syscall_event_set(struct rt_event *event) { rt_syscall_1(RT_SYSCALL_EVENT_SET, (uintptr_t)event);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_push`

```c
void rt_syscall_push(struct rt_syscall_record *record);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_syscall_pend`

```c
void rt_syscall_pend(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/syscall.h`
