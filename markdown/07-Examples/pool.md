# Example: `pool.c`

Blocking object pool + queue pipeline: getter allocates from a pool of `uint32_t` slots, tags them, pushes descriptors; putter pops, checks values, returns slots to the pool.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `getter` | 1 | `2 * RT_STACK_MIN` | `ITERATIONS` (1000): `rt_pool_get` → write index → `rt_queue_push`. |
| `putter` | 1 | `2 * RT_STACK_MIN` | Matching pops; assert `*e.p == i`; `rt_pool_put`; then `rt_exit`. |
| `timeout` | 0 | `RT_STACK_MIN` | Sleep 100; `rt_panic("timed out")`. |

## Shared state

```c
static uint32_t x[10];
RT_POOL_STATIC(pool, x);              /* pool over array x[10] */

struct elem { uint32_t *p; };
RT_QUEUE_STATIC(queue, struct elem, 5);
```

Pool capacity 10, queue depth 5 — getter can block on empty pool or full queue; putter blocks on empty queue.

## Control flow

1. Getter obtains a free `uint32_t *` from the pool (blocks if all 10 are outstanding).
2. Stores the loop index `i` into that slot, wraps the pointer in `struct elem`, pushes to the queue (blocks if 5 elements already queued).
3. Putter pops in order, asserts the payload equals the expected sequential `i`, returns the pointer to the pool.
4. After 1000 round-trips, putter `rt_exit`s.
5. Back-pressure from small pool/queue depth forces real blocking paths, not only the happy non-blocking case.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_POOL_STATIC` | Memory pool over static array `x`. |
| `rt_pool_get` / `rt_pool_put` | Allocate / free slots (blocking). |
| `RT_QUEUE_STATIC` | Bounded queue of `struct elem`. |
| `rt_queue_push` / `rt_queue_pop` | Handoff of pool pointers. |
| `rt_panic` | Watchdog. |

## Success / failure

- **Pass:** all asserts on `*e.p`, then `rt_exit`.
- **Fail:** value mismatch assert or timeout panic (deadlock / pool leak).

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/pool
```

## See also

- [Other Primitives](../03-Kernel-Features/Other-Primitives.md) · [Queues](../03-Kernel-Features/Queues.md)
- [queue](queue.md) · [API: pool](../09-API-Reference/pool.md)
- [Building and Examples](../07-Building-and-Examples.md)
