# Example: `empty.c`

Minimal single-task smoke test: one task drops privilege and exits.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `empty` | 0 | `RT_STACK_MIN` | Drops privilege, then calls `rt_exit()`. |

```c
static void empty(void)
{
    rt_task_drop_privilege();
    rt_exit();
}

RT_TASK(empty, RT_STACK_MIN, 0);
```

## Shared state

None. No mutexes, semaphores, or globals beyond the task itself.

## Control flow

1. The kernel starts and schedules the sole ready task `empty`.
2. `rt_task_drop_privilege()` leaves privileged mode (MPU / syscall path when enabled).
3. `rt_exit()` terminates the RT process / simulation cleanly.

There is no loop, handshake, or watchdog. Success is simply that the program reaches `rt_exit` without panicking.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_TASK` | Statically declare task `empty` at priority 0. |
| `rt_task_drop_privilege` | Leave privileged mode before exit. |
| `rt_exit` | End the example successfully. |

## Success / failure

- **Pass:** process exits via `rt_exit` with no panic.
- **Fail:** hang or panic would indicate a broken task start / exit path (unlikely in this tiny case).

## Build / run

Host signal port:

```bash
scons -j$(nproc)
./build/signal/examples/empty
```

## See also

- [Tasks](../03-Kernel-Features/Tasks.md)
- [simple](simple.md) — slightly richer two-task yield/exit demo
- [Building and Examples](../07-Building-and-Examples.md)
