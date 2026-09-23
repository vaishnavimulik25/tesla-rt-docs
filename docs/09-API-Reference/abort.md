# `rt/abort.h`

**Audience:** application.

## Functions

### `__attribute__`

```c
__attribute__((noreturn)) void rt_abort(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/abort.h`
