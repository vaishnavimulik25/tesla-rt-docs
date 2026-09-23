# Example: `rwlock.c`

Two readers and one writer share an rwlock: readers assert `x == y` under read locks; writer increments both under a write lock. A timeout exits after 100 ticks.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `reader` ×2 | 2 | `2 * RT_STACK_MIN` | Forever read-lock; assert `x == y`. |
| `writer` | 1 | `2 * RT_STACK_MIN` | Forever write-lock; `++x; ++y`. |
| `timeout` | 0 | `RT_STACK_MIN` | Sleep 100; `rt_exit`. |

Readers outrank the writer so they run preferentially when runnable, but still must not observe a torn `(x, y)` pair.

## Shared state

```c
static RT_RWLOCK(lock);
static unsigned x = 0;
static unsigned y = 0;
```

## Control flow

1. Writer updates `x` then `y` only while holding the write lock — readers must never see `x != y`.
2. Readers take `RT_RWLOCK_READ_GUARD` (shared) and assert equality.
3. After 100 ticks of concurrent activity, timeout exits successfully — pass means no assert fired during the window.

> Skipped for `cl2000`.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_RWLOCK` | Reader/writer lock object. |
| `RT_RWLOCK_READ_GUARD` | Scoped shared lock. |
| `RT_RWLOCK_WRITE_GUARD` | Scoped exclusive lock. |
| `rt_assert` | Consistency check. |

## Success / failure

- **Pass:** timeout `rt_exit` without `"x and y do not match"`.
- **Fail:** assert on mismatched x/y (broken exclusion) or hang.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/rwlock
```

## See also

- [Other Primitives](../03-Kernel-Features/Other-Primitives.md) · [API: rwlock](../09-API-Reference/rwlock.md)
- [mutex](mutex.md)
- [Building and Examples](../07-Building-and-Examples.md)
