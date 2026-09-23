# `rt/assert.h`

**Audience:** application.

## Functions

### `rt_assert`

```c
static inline void rt_assert(bool condition, const char *msg) { if (!condition) { rt_panic(msg);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/assert.h`
