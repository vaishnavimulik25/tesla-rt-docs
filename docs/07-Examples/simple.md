# Example: `simple.c`

Two equal-priority arg-tasks yield in a loop; the task with `arg == 1` exits the process.

## Tasks / roles

| Task | Arg | Priority | Stack | Role |
|------|-----|----------|-------|------|
| `simple` | `0` | 0 | `RT_STACK_MIN` | Yields 100 times, then returns (idle forever / descheduled). |
| `simple` | `1` | 0 | `RT_STACK_MIN` | Yields 100 times, then `rt_exit()`. |

```c
static void simple(uintptr_t arg)
{
    rt_task_drop_privilege();
    for (int i = 0; i < 100; ++i)
    {
        rt_task_yield();
    }

    if (arg == 1)
    {
        rt_exit();
    }
}

RT_TASK_ARG(simple, 0, RT_STACK_MIN, 0);
RT_TASK_ARG(simple, 1, RT_STACK_MIN, 0);
```

## Shared state

None. Tasks only share the scheduler; no synchronization objects.

## Control flow

1. Both tasks are equal priority (`0`), so they round-robin via `rt_task_yield`.
2. Each runs 100 yields after dropping privilege.
3. The instance with `arg == 1` calls `rt_exit()`, ending the whole example.
4. The `arg == 0` instance never exits on its own; process teardown from the peer is enough.

Priorities being equal matters: yield actually hands the CPU to the peer. With unequal priorities, a higher-priority spinner would starve the other without blocking.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_TASK_ARG` | Declare two instances of `simple` with different `uintptr_t` args. |
| `rt_task_drop_privilege` | Leave privileged mode. |
| `rt_task_yield` | Cooperative switch between equal-priority peers. |
| `rt_exit` | Successful process termination from `arg == 1`. |

## Success / failure

- **Pass:** after ~100 yield round-trips, `rt_exit` ends the run.
- **Fail:** hang (yield broken / wrong priorities) or never exiting (arg wiring wrong).

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/simple
```

## See also

- [Tasks](../03-Kernel-Features/Tasks.md) · [Scheduling](../03-Kernel-Features/Scheduling.md)
- [empty](empty.md) · [float](float.md) · [cycle-yield](cycle-yield.md)
- [Building and Examples](../07-Building-and-Examples.md)
