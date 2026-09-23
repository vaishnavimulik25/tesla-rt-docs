# Example: `notify.c`

Task notification object: notifier ORs a bit value `n` times, then posts; waiter clears-on-wait each time and finally asserts a bare wait times out.

## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `notifier` | 0 | `RT_STACK_MIN` | Sleep 5, `rt_notify_or(&note, 1)` × `n`; sleep 15; `rt_notify_post`. |
| `waiter` | 0 | `RT_STACK_MIN` | `rt_notify_timedwait_clear(..., expect 1, timeout 10)` × `n`; then assert plain timed wait fails; `rt_exit`. |

Equal priority; sleeps create the timing windows.

## Shared state

```c
static const int n = 10;
static RT_NOTIFY(note, 0);   /* initial notification value 0 */
```

## Control flow

1. Waiter blocks until the notification value matches / is signaled with clear semantics (`timedwait_clear` with expected value `1`).
2. Notifier ORs `1` into the note each iteration — wakes the clear-wait.
3. After `n` successes, waiter does `rt_notify_timedwait` (no clear helper) with 10 ticks and asserts **failure** while notifier is still in its 15-tick sleep before the final `post`.
4. `rt_exit` on the waiter; late `post` is unused.

Same shape as [sem](sem.md) but on the notify primitive (value-based rather than counting semaphore).

## APIs used

| API | Role in this example |
|-----|----------------------|
| `RT_NOTIFY` | Static notification object. |
| `rt_notify_or` | Bitwise-OR into the notify value. |
| `rt_notify_post` | Post/wake (final unused after exit path). |
| `rt_notify_timedwait_clear` | Wait for value, clear on success. |
| `rt_notify_timedwait` | Timed wait without the clear helper (negative test). |

## Success / failure

- **Pass:** `n` clear-waits succeed; next wait times out; `rt_exit`.
- **Fail:** timeout assert messages from `rt_assert`.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/notify
```

## See also

- [Notifications](../03-Kernel-Features/Notifications.md)
- [sem](sem.md) · [event](event.md) · [cycle-notify](cycle-notify.md)
- [Building and Examples](../07-Building-and-Examples.md)
