# `rt/sem.h`

**Audience:** application.

## Types

- `struct rt_sem`

## Macros

| Macro |
|-------|
| `RT_SEM_INIT_MAX` |
| `RT_SEM_INIT` |
| `RT_SEM_INIT_BINARY` |
| `RT_SEM_MAX` |
| `RT_SEM` |
| `RT_SEM_BINARY` |

## Functions

### `rt_sem_post`

```c
void rt_sem_post(struct rt_sem *sem);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_post_from_task`

```c
void rt_sem_post_from_task(struct rt_sem *sem);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_post_from_interrupt`

```c
void rt_sem_post_from_interrupt(struct rt_sem *sem);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_post_n`

```c
void rt_sem_post_n(struct rt_sem *sem, int n);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_post_n_from_task`

```c
void rt_sem_post_n_from_task(struct rt_sem *sem, int n);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_post_n_from_interrupt`

```c
void rt_sem_post_n_from_interrupt(struct rt_sem *sem, int n);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_wait`

```c
void rt_sem_wait(struct rt_sem *sem);
```

- **Blocking behavior:** May block
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_trywait`

```c
bool rt_sem_trywait(struct rt_sem *sem);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_timedwait`

```c
bool rt_sem_timedwait(struct rt_sem *sem, unsigned long ticks);
```

- **Blocking behavior:** Timed wait; returns status on timeout
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

### `rt_sem_add_n`

```c
void rt_sem_add_n(struct rt_sem *sem, int n);
```

- **Blocking behavior:** Non-blocking (or read-only)
- **ISR:** `post`/`post_n` ISR-safe (auto or `_from_interrupt`). Blocking wait not for ISR.
- **Examples:** `examples/sem.c`, `examples/water/sem.c`, `examples/cycle/sem.c`, `examples/fair.c`

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/sem.h`
