# Example: `tls.c`

Two tasks mutate independent `foo`/`bar` values — either via `_Thread_local` when `RT_TASK_LOCAL_STORAGE_ENABLE`, or separate statics as a fallback — asserting parity invariants across yields.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `task0` | 1 | `RT_STACK_MIN` | Init `foo0=1`; loop `foo0+=2`, yield, `bar0+=foo0`; assert `foo0` odd. |
| `task1` | 1 | `RT_STACK_MIN` | Init `foo1=0`; loop even updates; assert `foo1` and `bar1` even. |
| `timeout` | 0 | `RT_STACK_MIN` | Sleep 100; `rt_exit`. |

## Shared state

```c
#if RT_TASK_LOCAL_STORAGE_ENABLE
static _Thread_local uint32_t foo, bar = 2;
/* macros alias foo0/bar0/foo1/bar1 to the TLS symbols */
#else
static uint32_t foo0, bar0 = 2, foo1, bar1 = 2;
#endif
```

When TLS is enabled, each task sees its own `foo`/`bar` despite the same names — that is the point of the test. When disabled, separate globals simulate isolation.

## Control flow

1. Both workers yield frequently so the scheduler switches between them many times.
2. If TLS (or separate statics) works, task0’s odd `foo` never contaminates task1’s even invariants (and vice versa).
3. After 100 ticks, timeout exits — success is “no assert during the window”.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `_Thread_local` / TLS config | Per-task storage when enabled. |
| `rt_task_yield` | Force context switches. |
| `rt_assert` | Detect storage bleed. |
| `RT_TASK` | Two workers + timeout. |

## Success / failure

- **Pass:** timeout `rt_exit` without parity asserts failing.
- **Fail:** unexpected odd/even values → TLS not isolated.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/tls
```

## See also

- [Tasks](../03-Kernel-Features/Tasks.md) · [API: tls](../09-API-Reference/tls.md)
- [float](float.md)
- [Building and Examples](../07-Building-and-Examples.md)
