# `rt/start.h`

**Audience:** port / startup.

## Functions

### `__attribute__`

```c
__attribute__((noreturn)) void rt_start(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.
- **Examples:** `examples/simple.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/start.h`
