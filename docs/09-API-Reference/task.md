# `rt/task.h`

**Audience:** application.

## Types

- `struct rt_task`

## Enumerations

- `enum rt_task_state`

## Macros

| Macro |
|-------|
| `RT_TASK_CTX_OFFSET` |
| `RT_TASK_READY_CTZ_ENABLE` |
| `RT_TASK_READY_CTZ_ENABLE` |
| `RT_TASK_PRIORITY_MIN` |
| `RT_TASK_PRIORITY_MAX` |
| `RT_TASK_PRIORITY_IDLE` |
| `RT_TASK_CYCLE_ENABLE` |
| `RT_TASK_INIT` |
| `RT_CAT_` |
| `RT_CAT` |
| `RT_TASK_STACK_MPU_REGION` |
| `RT_TASK_MPU_CONFIG_INIT` |
| `RT_TASK_STACK_MPU_REGION` |
| `RT_TASK_MPU_CONFIG_INIT` |
| `RT_TASK_LOCAL_STORAGE_INIT` |
| `RT_TASK_LOCAL_STORAGE_MPU_REGION` |
| `RT_TASK_LOCAL_STORAGE_INIT` |
| `RT_TASK_LOCAL_STORAGE_MPU_REGION` |
| `RT_TASK_COMMON` |
| `RT_TASK` |
| `RT_TASK_ARG` |

## Functions

### `rt_task_yield`

```c
void rt_task_yield(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_sleep`

```c
void rt_task_sleep(unsigned long ticks);
```

- **Blocking behavior:** May block
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_sleep_periodic`

```c
void rt_task_sleep_periodic(unsigned long *last_wake_tick, unsigned long period);
```

- **Blocking behavior:** May block
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `__attribute__`

```c
__attribute__((noreturn)) void rt_task_exit(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_drop_privilege`

```c
void rt_task_drop_privilege(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_init`

```c
void rt_task_init(struct rt_task *task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_pend_terminate`

```c
bool rt_task_pend_terminate(struct rt_task *task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_pend_restart`

```c
bool rt_task_pend_restart(struct rt_task *task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_terminate`

```c
void rt_task_terminate(struct rt_task *task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_restart`

```c
void rt_task_restart(struct rt_task *task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_tls_init`

```c
rt_tls_init(fn##_task_local_storage, sizeof fn##_task_local_storage);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `rt_task_init`

```c
rt_task_init(&fn_##_task);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

### `__attribute__`

```c
__attribute__((noreturn)) void rt_task_entry(void);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** yield/sleep/exit are task context. pend_terminate/restart may be used more broadly — see source.
- **Examples:** `examples/simple.c`, `examples/empty.c`, `examples/fair.c`, `examples/sleep.c`, `examples/float.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/task.h`
