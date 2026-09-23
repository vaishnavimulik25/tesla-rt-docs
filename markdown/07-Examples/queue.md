# Example: `queue.c`

Two pushers and one popper on a static MPMC-style queue; popper peeks then pops and checks per-pusher monotonic order until a timeout exits.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `popper` | 1 | `2 * RT_STACK_MIN` | Forever: peek, pop, assert equal; track max per pusher. |
| `pusher` arg `0` | 1 | `2 * RT_STACK_MIN` | Forever push increasing values in band 0. |
| `pusher` arg `TASK_INC` (`0x1000000`) | 1 | `2 * RT_STACK_MIN` | Same for band 1. |
| `timeout` | 0 | `RT_STACK_MIN` | Sleep 1000 ticks; `rt_exit` (success by duration). |

Encoding: `x = task_band + elem`; `task = x / TASK_INC`, `elem = x % TASK_INC`.

## Shared state

```c
RT_QUEUE_STATIC(queue, uint32_t, 10);
static uint32_t max_elem[NPUSHERS];
static rt_atomic_size_t num_popped;
```

## Control flow

1. Each pusher starts at its band base and increments forever, blocking on full queue when needed.
2. Popper `rt_queue_peek`s then `pop`s — asserts peek matches pop (no torn observation).
3. For each value, asserts `elem >= max_elem[task]` so a single pusher’s stream stays ordered even when interleaved with the other.
4. After 1000 ticks the timeout task `rt_exit`s — this example’s “pass” is surviving the stress window without assert failures (not a fixed message count).

Equal priorities mean push/pop progress depends on fair scheduling + blocking on empty/full.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_QUEUE_STATIC` | Bounded `uint32_t` queue (depth 10). |
| `rt_queue_push` / `pop` / `peek` | Produce / consume / non-destructive read. |
| `RT_TASK_ARG` | Two pushers with different base args. |
| `rt_assert` | Ordering and peek/pop consistency. |

## Success / failure

- **Pass:** run until timeout `rt_exit` with no asserts.
- **Fail:** peek≠pop, out-of-order per task, or hang (less likely with blocking queue).

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/queue
```

## See also

- [Queues](../03-Kernel-Features/Queues.md)
- [pool](pool.md) · [cycle-queue](cycle-queue.md)
- [Building and Examples](../07-Building-and-Examples.md)
