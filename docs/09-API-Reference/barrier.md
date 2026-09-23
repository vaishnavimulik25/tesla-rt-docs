# `rt/barrier.h`

**Audience:** application.

## Types

- `struct rt_barrier`

## Macros

| Macro |
|-------|
| `RT_BARRIER_INIT` |
| `RT_BARRIER` |

## Functions

### `rt_barrier_wait`

```c
bool rt_barrier_wait(struct rt_barrier *barrier);
```

- **Blocking behavior:** May block
- **ISR:** Task-only (mutex/cond).
- **Examples:** `examples/water/barrier.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/barrier.h`
