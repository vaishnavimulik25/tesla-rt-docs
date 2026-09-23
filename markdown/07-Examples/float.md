# Example: `float.c`

Two tasks perform floating-point accumulate-and-yield loops to exercise FP context save/restore; a timeout exits after 100 ticks.

## Tasks / roles

| Task | Arg | Priority | Stack | Role |
|------|-----|----------|-------|------|
| `f` | `1` | 1 | `RT_STACK_FP_MIN` | Forever: `x += 1.0f`, store to `v`, yield. |
| `f` | `2` | 1 | `RT_STACK_FP_MIN` | Same with addend 2.0f. |
| `timeout` | — | 0 | `RT_STACK_MIN` | Sleep 100; `rt_exit`. |

Stacks use `RT_STACK_FP_MIN` because FP contexts may need more stack than `RT_STACK_MIN` on some arches.

## Shared state

```c
static volatile float v;  /* last written FP value; races are intentional */
```

No synchronization — the test is that FP registers / lazy FP state survive switches without crashing or corrupting control flow.

## Control flow

1. Both FP tasks run at equal priority, yielding every iteration.
2. Each keeps a private `float x` on its stack and writes through volatile `v`.
3. Timeout ends the stress window successfully if nothing faulted.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_TASK_ARG` | Two FP workers with different addends. |
| `RT_STACK_FP_MIN` | Stack large enough for FP use. |
| `rt_task_yield` | Context switch with live FP state. |
| `rt_exit` | End after timeout. |

## Success / failure

- **Pass:** timeout `rt_exit` (no FP exception / hang).
- **Fail:** crash or hang if FP context switch is broken on the port.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/float
```

## See also

- [Tasks](../03-Kernel-Features/Tasks.md) · [Architecture and Ports](../06-Architecture-and-Ports.md)
- [tls](tls.md) · [simple](simple.md)
- [Building and Examples](../07-Building-and-Examples.md)
