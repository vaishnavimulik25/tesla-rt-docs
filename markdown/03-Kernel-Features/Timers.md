# Software Timers

Header: `include/rt/timer.h`. Implementation: `src/timer.c`.

Timers are **not** IRQ callbacks by themselves: commands go through a queue consumed by **`rt_timer_daemon`**, which you run as a task via `RT_TIMER_TASK`.

## Setup pattern

```c
RT_TIMER_QUEUE_STATIC(queue, 4);
RT_TIMER_TASK(&queue, 2 * RT_STACK_MIN, 0);  // daemon task

static void f(void) { /* ... */ }
static RT_TIMER_PERIODIC(timer, &queue, f, 5);

rt_timer_start(&timer);
/* later */ rt_timer_stop(&timer);
rt_timer_change_period(&timer, 3);
```

## Macros

| Macro | Meaning |
|-------|---------|
| `RT_TIMER_QUEUE` / `_STATIC` | Queue of `struct rt_timer_command` |
| `RT_TIMER_TASK(queue, stack, prio, …)` | `RT_TASK_ARG(rt_timer_daemon, queue, …)` |
| `RT_TIMER_ONE_SHOT` / `_ARG` | One-shot timer object |
| `RT_TIMER_PERIODIC` / `_ARG` | Periodic timer object |
| `RT_TIMER_AUTOSTART(timer)` | Constructor calls `rt_timer_trystart` |

## Control API

Each of start / stop / change_period has blocking, try, and timed variants:

- `rt_timer_start` / `trystart` / `timedstart`
- `rt_timer_stop` / `trystop` / `timedstop`
- `rt_timer_change_period` / `trychange_period` / `timedchange_period`

Try/timed variants return `bool` (queue space / success).

## Commands

```c
enum rt_timer_op { RT_TIMER_OP_START, RT_TIMER_OP_STOP, RT_TIMER_OP_CHANGE_PERIOD };
```

## Example

`examples/timer.c` — start/stop/change period with assertions on callback counts.
