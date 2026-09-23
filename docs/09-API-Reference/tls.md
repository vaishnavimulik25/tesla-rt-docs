# `rt/tls.h`

**Audience:** application.

## Macros

| Macro |
|-------|
| `RT_TASK_LOCAL_STORAGE_ENABLE` |
| `RT_TASK_LOCAL_STORAGE_SIZE` |
| `RT_TLS_SECTION` |
| `RT_TLS` |

## Functions

### `rt_tls_set`

```c
void rt_tls_set(void *tls);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** See semantics; avoid blocking calls in ISRs.
- **Examples:** `examples/tls.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/tls.h`
