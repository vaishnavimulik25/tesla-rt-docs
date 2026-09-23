# `rt/trap.h`

**Audience:** application / tests.

## Functions

### `__attribute__`

```c
__attribute__((noreturn)) void rt_trap(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.
- **Examples:** `examples/cycle/yield.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/trap.h`
