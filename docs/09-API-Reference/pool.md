# `rt/pool.h`

**Audience:** application.

## Types

- `struct rt_pool`

## Macros

| Macro |
|-------|
| `RT_POOL_ARRAY_COUNT` |
| `RT_POOL_INIT` |
| `RT_POOL_COMMON` |
| `RT_POOL` |
| `RT_POOL_STATIC` |

## Functions

### `rt_pool_put`

```c
void rt_pool_put(struct rt_pool *, void *);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Blocking get task-only.
- **Examples:** `examples/pool.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/pool.h`
