# Queues

Header: `include/rt/queue.h`. Implementation: `src/queue.c`.

## Purpose and when to use

`rt_queue` is a **bounded, multi-producer, multi-consumer** queue of fixed-size elements. It supports blocking, timed, and non-blocking push, pop, and peek. Use it when:

- Tasks (and carefully designed ISR producers using try/timed-0 paths via underlying semaphores) exchange messages by value.
- You need a lock-free slot protocol with semaphore-backed empty/full waits.

From the header comment: as many concurrent accessors as there are **slots** may operate at once; additional operations block, time out, or fail until a prior operation completes.

## Data type and static initialization

```c
struct rt_queue {
    struct rt_sem push_sem;    /* space available */
    struct rt_sem pop_sem;     /* items available */
    rt_atomic_size_t enq, deq;
    rt_atomic_uchar *slots;
    void *data;
    size_t num_elems, elem_size;
};
```

```c
RT_QUEUE(name, type, num);          /* file-scope storage for slots + data */
RT_QUEUE_STATIC(name, type, num);   /* static storage duration */

/* Manual init if you provide buffers: */
struct rt_queue q = RT_QUEUE_INIT(q, type, num, slots_buf, data_buf);
```

`num` must be `<= RT_QUEUE_MAX_SIZE` (derived from `size_t` width and generation bits). Helpers `rt_queue_qgen` / `rt_queue_qindex` / `rt_queue_qsgen` are for the lock-free slot protocol — **not typical application API**.

## Public API

```c
void rt_queue_push(struct rt_queue *queue, const void *elem);
void rt_queue_pop(struct rt_queue *queue, void *elem);
void rt_queue_peek(struct rt_queue *queue, void *elem);

bool rt_queue_trypush(struct rt_queue *queue, const void *elem);
bool rt_queue_trypop(struct rt_queue *queue, void *elem);
bool rt_queue_trypeek(struct rt_queue *queue, void *elem);

bool rt_queue_timedpush(struct rt_queue *queue, const void *elem,
                        unsigned long ticks);
bool rt_queue_timedpop(struct rt_queue *queue, void *elem, unsigned long ticks);
bool rt_queue_timedpeek(struct rt_queue *queue, void *elem,
                        unsigned long ticks);
```

| Family | Empty/full behavior |
|--------|---------------------|
| `push` / `pop` / `peek` | Block until space / data |
| `try*` | Return `false` immediately if cannot proceed |
| `timed*` | Return `false` on timeout |

`peek` copies the front element **without** removing it (still requires an available item, like pop).

Elements are copied by `elem_size` (`sizeof(type)` in the macros).

## Priority donation

None on the queue object itself. Blocking uses the internal push/pop semaphores (priority-ordered waiters).

## ISR-safety rules

Blocking push/pop/peek must not run in an ISR. Non-blocking `try*` may be used where the underlying semaphore try paths are safe; for ISR producers, prefer `trypush` and handle `false` (e.g. drop or pend a function). Prefer documenting board-specific ISR→task patterns in your port.

## Common pitfalls

1. Passing a pointer to a temporary that goes out of scope before a *blocking* push copies it — push copies immediately on success, but ensure `elem` remains valid for the duration of the call.
2. Oversized queues — limited by `RT_QUEUE_MAX_SIZE`.
3. Assuming peek unblocks consumers — it does not remove the item.
4. Concurrent peek + pop races — both may see the same element; that is by design of MPMC peek.
5. Using queue helpers (`qgen`/`qindex`) in app code — internal.

## Related examples

| Example | Demonstrates |
|---------|----------------|
| `examples/queue.c` | multi-pusher / popper |
| `examples/timer.c` | timer command queue (`RT_TIMER_QUEUE_*`) |
| `examples/cycle/queue.c` | push→pop cycle latency |
| `examples/pool.c` | pool + queue of pointers/objects |
