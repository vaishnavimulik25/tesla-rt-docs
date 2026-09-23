# Tasks

Primary header: `include/rt/task.h` in the `rt` tree (`/workspace/rtng-rt`).

## Priority convention

| Macro | Value | Meaning |
|-------|-------|---------|
| `RT_TASK_PRIORITY_MIN` | `0` | **Highest** priority |
| `RT_TASK_PRIORITY_MAX` | `31` | Lowest numeric bound |
| `RT_TASK_PRIORITY_IDLE` | `31` | Idle task priority |

The scheduler selects the ready task with the **minimum** priority number (`min_ready_priority` / CTZ of `rt_ready_bits` in `src/rt.c`). Lower number ⇒ runs first.

Each task stores `priority` (effective, may rise via donation) and `base_priority` (configured).

## Creating tasks

### `RT_TASK` / `RT_TASK_ARG`

```c
RT_TASK(fn, stack_size, priority, /* optional MPU regions */);
RT_TASK_ARG(fn, arg, stack_size, priority, /* optional MPU regions */);
```

These macros (constructor + `used`) allocate a static stack and `struct rt_task`, call `rt_context_init`, then `rt_task_init`. They may only effectively register tasks **before** `rt_start`.

- `fn` is `void(void)` or `void(uintptr_t)` depending on macro.
- Name string defaults to `#fn` or `#fn(#arg)`.
- Priority is static-asserted `<= RT_TASK_PRIORITY_MAX`.

### Manual init

```c
void rt_task_init(struct rt_task *task);  // only before rt_start
```

`RT_TASK_INIT(name_, name_str, priority_)` initializes list fields / state for a manually declared TCB; you still must set `ctx`, `fn`, stack fields, etc., as `RT_TASK_COMMON` does.

## Task states (`enum rt_task_state`)

| State | Meaning |
|-------|---------|
| `RT_TASK_STATE_READY` | Runnable / running |
| `RT_TASK_STATE_BLOCKED_ON_SEM_WAIT` | Waiting on semaphore |
| `RT_TASK_STATE_BLOCKED_ON_SEM_TIMEDWAIT` | Timed sem wait |
| `RT_TASK_STATE_BLOCKED_ON_MUTEX_LOCK` | Waiting on mutex |
| `RT_TASK_STATE_BLOCKED_ON_MUTEX_TIMEDLOCK` | Timed mutex lock |
| `RT_TASK_STATE_BLOCKED_ON_EVENT_WAIT` | Waiting on event |
| `RT_TASK_STATE_BLOCKED_ON_EVENT_TIMEDWAIT` | Timed event wait |
| `RT_TASK_STATE_ASLEEP` | Sleeping |
| `RT_TASK_STATE_EXITED` | Exited |
| `RT_TASK_STATE_TERMINATED` | Terminated |

## Runtime APIs

| API | Description |
|-----|-------------|
| `rt_task_yield()` | Yield to another task at the **same** priority (if still highest, continues) |
| `rt_task_sleep(ticks)` | Sleep for ticks (`0` = no-op) |
| `rt_task_sleep_periodic(&last, period)` | Sleep until `*last + period`; updates `*last` |
| `rt_task_exit()` | Exit current task (`noreturn`); returning from task fn also exits |
| `rt_task_name()` | Current task name |
| `rt_task_self()` | Pointer to current `struct rt_task` |
| `rt_task_drop_privilege()` | Make current task unprivileged (no-op on some arch) |
| `rt_task_pend_terminate(task)` | Enqueue terminate; `false` if already terminating/restarting |
| `rt_task_pend_restart(task)` | Enqueue restart |
| `rt_task_terminate` / `rt_task_restart` | **Syscall-handler only** |

## Idle task

The kernel defines:

```c
RT_TASK(rt_idle, RT_STACK_MIN, RT_TASK_PRIORITY_IDLE);
```

`rt_idle()` is declared in `rt/idle.h`. It runs when no higher-priority task is ready.

## Globals

- `struct rt_task *rt_active_task` — current task  
- `struct rt_list rt_global_task_list` — all tasks  

## Example

See `examples/simple.c` and [Getting Started](../02-Getting-Started.md).
