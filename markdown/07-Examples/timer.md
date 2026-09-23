# Example: `timer.c`

Software timer daemon: a periodic timer increments an atomic counter; the user task starts/stops it and changes the period, asserting expected fire counts.

## Tasks / roles

| Task / object | Priority | Stack | Role |
|---------------|----------|-------|------|
| Timer daemon (`RT_TIMER_TASK`) | (macro) | `2 * RT_STACK_MIN` | Drains `queue`, runs timer callbacks. |
| `task0` | 0 | `2 * RT_STACK_MIN` | Starts/stops/changes period; asserts counts; `rt_exit`. |
| Callback `f` | — | — | `rt_atomic_fetch_add(&x, 1)`. |

## Shared state

```c
RT_TIMER_QUEUE_STATIC(queue, 4);
RT_TIMER_TASK(&queue, 2 * RT_STACK_MIN, 0);
static rt_atomic_int x = 0;
static RT_TIMER_PERIODIC(timer, &queue, f, 5);  /* period 5 ticks */
```

## Control flow

1. `rt_timer_start` — timer fires every 5 ticks via the daemon.
2. Sleep 18 ticks → expect **3** firings (`18/5` → 3), assert `x == 3`, then `rt_timer_stop`.
3. `rt_timer_change_period(&timer, 3)`; start again; sleep 25 → additional fires: `(11 - 3) = 8` more if arithmetic matches source expect `x == 11` total.
4. Stop and `rt_exit`.

Exact counts depend on when the first deadline is scheduled relative to start; the asserts encode the kernel’s observed semantics for this port.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_TIMER_QUEUE_STATIC` | Queue backing the timer daemon. |
| `RT_TIMER_TASK` | Daemon task declaration. |
| `RT_TIMER_PERIODIC` | Periodic timer object + callback. |
| `rt_timer_start` / `stop` / `change_period` | Lifecycle under test. |
| `rt_atomic_*` | Count callback invocations. |

## Success / failure

- **Pass:** both count asserts succeed; `rt_exit`.
- **Fail:** `"x has an unexpected value"`.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/timer
```

## See also

- [Timers](../03-Kernel-Features/Timers.md)
- [sleep](sleep.md)
- [Building and Examples](../07-Building-and-Examples.md)
