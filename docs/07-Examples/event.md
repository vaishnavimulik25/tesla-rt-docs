# Example: `event.c`

Multi-bit event object: a setter pulses bits A/B/C; waiters exercise **ANY+NOCLEAR** and **ALL** waits, including a final intentional timeout.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `setter` | 2 | `RT_STACK_MIN` | For `n=10`: sleep/set A, sleep/set B, sleep/set C; finally set A\|B\|C together. |
| `waiter_all` | 1 | `RT_STACK_MIN` | Wait ALL of A\|B\|C (clearing) `n` times; then assert a short wait times out; `rt_exit`. |
| `waiter_one_noclear` | 0 | `RT_STACK_MIN` | Wait ANY of B\|D with `RT_EVENT_WAIT_NOCLEAR`, `n` times. |

## Shared state

```c
static const int n = 10;
static RT_EVENT(event);
#define EVENT_A (1u << 0)
#define EVENT_B (1u << 1)
#define EVENT_C (1u << 2)
#define EVENT_D (1u << 3)
```

## Control flow

1. **waiter_all** blocks until A, B, and C are all set (`RT_EVENT_WAIT_ALL`). Default clear semantics consume matched bits on wake.
2. **setter** spaces individual sets so ALL waiters must accumulate bits across posts within the 15-tick timed wait.
3. **waiter_one_noclear** waits for B (or D) with `RT_EVENT_WAIT_NOCLEAR` — matched bits remain set so other waiters / subsequent checks still see them.
4. After `n` successful ALL waits, setter’s final burst and a short 10-tick wait: `waiter_all` asserts the wait **does not** match (timeout / incomplete bits) via `!rt_event_bits_match`, then exits.
5. Priorities: setter highest so bit production keeps pace; ALL waiter above NOCLEAR waiter.

`rt_event_bits_match(bits, wait)` interprets the wait flags embedded in the mask.

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_EVENT` | Bitmask event object. |
| `rt_event_set` | OR bits into the event. |
| `rt_event_timedwait` | Wait with timeout; returns observed bits. |
| `RT_EVENT_WAIT_ALL` / `RT_EVENT_WAIT_NOCLEAR` | Wait mode flags in the mask. |
| `rt_event_bits_match` | Test whether wait condition was satisfied. |

## Success / failure

- **Pass:** `n` successful waits of each style; final ALL wait times out as asserted; `rt_exit`.
- **Fail:** `"wait timed out"` / `"wait didn't time out"` asserts.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/event
```

## See also

- [Events](../03-Kernel-Features/Events.md)
- [notify](notify.md) · [cycle-event](cycle-event.md)
- [Building and Examples](../07-Building-and-Examples.md)
