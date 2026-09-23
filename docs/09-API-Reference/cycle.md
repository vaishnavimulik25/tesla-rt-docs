# `rt/cycle.h`

**Audience:** application / benchmarks.

## Macros

| Macro |
|-------|
| `RT_CYCLE_ENABLE` |

## Functions

### `rt_cycle_init`

```c
void rt_cycle_init(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Read cycle counter; pause/resume noted as syscall-handler-only in header.
- **Examples:** `examples/cycle/yield.c`, `examples/cycle/mutex.c`, `examples/cycle/sem.c`

### `rt_cycle`

```c
uint32_t rt_cycle(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Read cycle counter; pause/resume noted as syscall-handler-only in header.
- **Examples:** `examples/cycle/yield.c`, `examples/cycle/mutex.c`, `examples/cycle/sem.c`

### `rt_task_cycle_pause`

```c
void rt_task_cycle_pause(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Read cycle counter; pause/resume noted as syscall-handler-only in header.
- **Examples:** `examples/cycle/yield.c`, `examples/cycle/mutex.c`, `examples/cycle/sem.c`

### `rt_task_cycle_resume`

```c
void rt_task_cycle_resume(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Read cycle counter; pause/resume noted as syscall-handler-only in header.
- **Examples:** `examples/cycle/yield.c`, `examples/cycle/mutex.c`, `examples/cycle/sem.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/cycle.h`
