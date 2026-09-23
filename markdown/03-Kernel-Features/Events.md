# Events (Event Flags)

Header: `include/rt/event.h`. Implementation: `src/event.c`.

## Purpose and when to use

An event object holds a set of **bit flags** that tasks can wait for (ANY or ALL), optionally clearing them on wake. Use events when:

- Multiple conditions must be combined (e.g. “A or B”, “A and B and C”).
- An ISR needs to set flags for a waiting task without a queue payload.
- You want FreeRTOS-style event groups / Zephyr-style event bits.

## Data type and static initialization

```c
struct rt_event {
    rt_atomic_uint32_t bits;
    struct rt_list wait_list;
    struct rt_syscall_record set_record; /* ISR-safe pendable set */
};

RT_EVENT(name);
struct rt_event e = RT_EVENT_INIT(e);
```

### Wait option flags (OR into the wait mask)

| Constant | Role |
|----------|------|
| Bits `0..28` | Application event bits (`rt_event_bits()` masks reserved bits) |
| `RT_EVENT_WAIT_ALL` | Wait until **all** requested bits are set (otherwise ANY) |
| `RT_EVENT_WAIT_NOCLEAR` | Do not clear matched bits on successful wait |
| `RT_EVENT_WAITED_MASK` | Internal: waiters present |
| `RT_EVENT_WAIT_RESERVED` | Mask of reserved high bits |

```c
static inline uint32_t rt_event_bits(uint32_t bits);
bool rt_event_bits_match(uint32_t bits, uint32_t wait);
```

`rt_event_bits_match` returns whether `bits` satisfies the ANY/ALL condition encoded in `wait`.

## Public API

```c
uint32_t rt_event_get(const struct rt_event *event);
uint32_t rt_event_clear(struct rt_event *event, uint32_t bits);
uint32_t rt_event_set(struct rt_event *event, uint32_t bits);
uint32_t rt_event_set_from_task(struct rt_event *event, uint32_t bits);
uint32_t rt_event_set_from_interrupt(struct rt_event *event, uint32_t bits);

uint32_t rt_event_wait(struct rt_event *event, uint32_t wait);
uint32_t rt_event_trywait(struct rt_event *event, uint32_t wait);
uint32_t rt_event_timedwait(struct rt_event *event, uint32_t wait,
                            unsigned long ticks);
```

| Function | Blocks? | Returns | Notes |
|----------|---------|---------|-------|
| `rt_event_get` | No | Current bit word | Non-destructive read. |
| `rt_event_clear` | No | Previous bits | Clears the given application bits. |
| `rt_event_set` | No | Previous bits | Auto task/ISR path (like sem post). |
| `rt_event_set_from_task` | No | Previous bits | Task/syscall path when waiters exist. |
| `rt_event_set_from_interrupt` | No | Previous bits | Uses `set_record` when needed. |
| `rt_event_wait` | Yes | Bits that satisfied the wait (see impl) | Clears matched bits unless `NOCLEAR`. |
| `rt_event_trywait` | No | `0` if not satisfied, else matching bits | Non-blocking. |
| `rt_event_timedwait` | Up to `ticks` | `0` on timeout, else matching bits | |

Typical wait mask:

```c
uint32_t w = EVENT_A | EVENT_B | RT_EVENT_WAIT_ALL;           /* need both */
uint32_t x = EVENT_B | RT_EVENT_WAIT_NOCLEAR;                 /* any, keep bits */
uint32_t bits = rt_event_timedwait(&event, w, 15);
rt_assert(rt_event_bits_match(bits, w), "wait timed out");
```

## Priority donation

No owner / no inheritance. Waiters are queued by priority for wake processing when bits are set.

## ISR-safety rules

| API | Task | ISR |
|-----|------|-----|
| `get` / `clear` / `trywait` | Yes | Yes (non-blocking) |
| `set` / `set_from_interrupt` | Yes | Yes |
| `wait` / `timedwait` | Yes | **No** |

## Common pitfalls

1. Setting reserved high bits as application flags — use only bits below the reserved field; `rt_event_bits()` strips them.
2. Forgetting `RT_EVENT_WAIT_ALL` when you need conjunction.
3. Expecting `trywait`/`timedwait` to return the full register — check with `rt_event_bits_match`.
4. Clearing bits from one waiter while another needed `NOCLEAR`.
5. Blocking wait from an ISR.

## Related examples

| Example | Demonstrates |
|---------|----------------|
| `examples/event.c` | ANY/ALL, NOCLEAR, timedwait |
| `examples/cycle/event.c` | set→wait cycle latency |
