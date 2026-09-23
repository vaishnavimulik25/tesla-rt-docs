# `rt/once.h`

**Audience:** application.

## Types

- `struct rt_once`

## Macros

| Macro |
|-------|
| `RT_ONCE_INIT` |
| `RT_ONCE` |

## Functions

### `rt_once_call`

```c
void rt_once_call(struct rt_once *once, void (*fn)(void));
```

- **Blocking behavior:** May block
- **ISR:** Task-only.
- **Examples:** `examples/once.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/once.h`
