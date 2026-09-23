# `rt/tick.h`

**Audience:** application / port.

## Functions

### `rt_tick_advance`

```c
void rt_tick_advance(void);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `rt_tick_advance` from tick ISR/port. `rt_tick_count` readable broadly.
- **Examples:** `examples/sleep.c`, `examples/cycle/sleep.c`

### `rt_tick_count`

```c
unsigned long rt_tick_count(void);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `rt_tick_advance` from tick ISR/port. `rt_tick_count` readable broadly.
- **Examples:** `examples/sleep.c`, `examples/cycle/sleep.c`

### `rt_tick_elapse`

```c
static inline void rt_tick_elapse(unsigned long *ticks, unsigned long *start_tick) { const unsigned long now = rt_tick_count();
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `rt_tick_advance` from tick ISR/port. `rt_tick_count` readable broadly.
- **Examples:** `examples/sleep.c`, `examples/cycle/sleep.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/tick.h`
