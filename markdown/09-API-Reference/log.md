# `rt/log.h`

**Audience:** application.

## Macros

| Macro |
|-------|
| `RT_LOG_ENABLE` |

## Functions

### `__attribute__`

```c
__attribute__((format(printf, 1, 2))) void rt_logf(const char *format, ...);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_log_flush`

```c
void rt_log_flush(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/log.h`
