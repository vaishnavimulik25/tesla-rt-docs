# `rt/timer.h`

**Audience:** application.

## Types

- `struct rt_timer`
- `struct rt_timer_command`

## Enumerations

- `enum rt_timer_op`

## Macros

| Macro |
|-------|
| `RT_TIMER_QUEUE` |
| `RT_TIMER_QUEUE_STATIC` |
| `RT_TIMER_TASK` |
| `RT_TIMER_INIT` |
| `RT_TIMER_INIT_ARG` |
| `RT_TIMER_ONE_SHOT` |
| `RT_TIMER_ONE_SHOT_ARG` |
| `RT_TIMER_PERIODIC` |
| `RT_TIMER_PERIODIC_ARG` |
| `RT_TIMER_AUTOSTART` |

## Functions

### `rt_timer_start`

```c
void rt_timer_start(struct rt_timer *timer);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_trystart`

```c
bool rt_timer_trystart(struct rt_timer *timer);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_timedstart`

```c
bool rt_timer_timedstart(struct rt_timer *timer, unsigned long block_ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_stop`

```c
void rt_timer_stop(struct rt_timer *timer);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_trystop`

```c
bool rt_timer_trystop(struct rt_timer *timer);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_timedstop`

```c
bool rt_timer_timedstop(struct rt_timer *timer, unsigned long block_ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_change_period`

```c
void rt_timer_change_period(struct rt_timer *timer, unsigned long period);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_trychange_period`

```c
bool rt_timer_trychange_period(struct rt_timer *timer, unsigned long period);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_timedchange_period`

```c
bool rt_timer_timedchange_period(struct rt_timer *timer, unsigned long period, unsigned long block_ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_timer_daemon`

```c
void rt_timer_daemon(uintptr_t arg);
```

- **Blocking behavior:** May block
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

### `rt_assert`

```c
rt_assert(rt_timer_trystart(&(timer)), "can't autostart timer");
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** start/stop/change go through a queue to the timer daemon task; see try/timed variants.
- **Examples:** `examples/timer.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/timer.h`
