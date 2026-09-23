# `rt/pend_function.h`

**Audience:** application / ISR.

## Macros

| Macro |
|-------|
| `RT_PEND_FUNCTION_RECORD` |

## Functions

### `rt_pend_function`

```c
bool rt_pend_function(struct rt_syscall_record *record, void (*fn)(uintptr_t), uintptr_t arg);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** Designed for ISR→handler deferral.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/pend_function.h`
