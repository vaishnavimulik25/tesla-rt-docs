# Example: `recursive.c`

Same-task nested locking with a recursive mutex: `foo` holds the lock while calling `bar`, which locks again.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `locker` | 0 | `RT_STACK_MIN` | Drops privilege, calls `foo()`, then `rt_exit`. |

Helpers (same task context, not separate `RT_TASK`s):

- `foo` — takes recursive mutex, `++x`, calls `bar`.
- `bar` — takes the same mutex again, `x *= 2`.

## Shared state

```c
static RT_MUTEX_RECURSIVE(mutex);
static unsigned x = 0;
```

## Control flow

1. `locker` enters `foo`, which acquires the recursive mutex via `RT_MUTEX_GUARD`.
2. Still holding the lock, `foo` calls `bar`.
3. `bar` acquires the **same** mutex again (allowed only because it is recursive). A non-recursive mutex would deadlock here.
4. Nested unlocks via cleanup restore the count; final `x` is `1 * 2 = 2` conceptually (start 0 → ++ → *=2).
5. `locker` calls `rt_exit`.

No watchdog task — the scenario is tiny and single-threaded from the scheduler’s view after start.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_MUTEX_RECURSIVE` | Mutex that allows same-owner re-entry. |
| `RT_MUTEX_GUARD` | Scoped lock/unlock at each nesting level. |
| `rt_task_drop_privilege` / `rt_exit` | Privilege drop and clean exit. |

## Success / failure

- **Pass:** nested locks complete without deadlock; `rt_exit`.
- **Fail:** hang (if mutex were non-recursive) or crash on unlock imbalance.

> Skipped for `cl2000` (cleanup attribute).

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/recursive
```

## See also

- [Mutexes](../03-Kernel-Features/Mutexes.md)
- [mutex](mutex.md) · [donate](donate.md)
- [Building and Examples](../07-Building-and-Examples.md)
