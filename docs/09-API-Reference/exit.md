# `rt/exit.h`

**Audience:** application.

## Functions

### `__attribute__`

```c
__attribute__((noreturn)) void rt_exit(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.
- **Examples:** `examples/empty.c`, `examples/simple.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/exit.h`
