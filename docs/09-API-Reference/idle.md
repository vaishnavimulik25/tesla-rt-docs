# `rt/idle.h`

**Audience:** port / application.

## Functions

### `rt_idle`

```c
void rt_idle(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/idle.h`
