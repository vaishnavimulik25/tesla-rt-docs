# `rt/list.h`

**Audience:** internal.

!!! warning "Internal / advanced"
    Intended for ports, tracing, or kernel internals. Application code should prefer task/sync/timer APIs.

## Types

- `struct rt_list`

## Macros

| Macro |
|-------|
| `RT_LIST_INIT` |
| `RT_LIST` |

## Functions

### `rt_list_remove`

```c
void rt_list_remove(struct rt_list *node);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_list_is_empty`

```c
bool rt_list_is_empty(const struct rt_list *list);
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_list_push_back`

```c
void rt_list_push_back(struct rt_list *list, struct rt_list *node);
```

- **Blocking behavior:** May block
- **ISR:** See semantics; avoid blocking calls in ISRs.

### `rt_list_insert_by`

```c
void rt_list_insert_by(struct rt_list *list, struct rt_list *node, bool (*cmp)(const struct rt_list *, const struct rt_list *));
```

- **Blocking behavior:** See description / typically non-blocking
- **ISR:** See semantics; avoid blocking calls in ISRs.

## See also

- Kernel feature docs under `03-Kernel-Features/` when applicable
- Source: `include/rt/list.h`
