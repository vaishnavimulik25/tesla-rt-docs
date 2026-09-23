# Scheduling

Behavior is implemented primarily in `src/rt.c`, driven by syscalls and tick.

## Preemptive, priority-based

- Ready tasks are indexed by priority bit in `rt_ready_bits` and per-priority circular lists `rt_ready_lists[0..31]`.
- On every scheduling decision, the kernel picks the ready list for `min_ready_priority()` — the **lowest numeric** priority with a ready task.
- A higher-priority task becoming ready preempts a lower-priority runner when the syscall/pendable/tick path returns a new task from `sched()`.

## Same-priority fairness

`rt_task_yield()` issues `RT_SYSCALL_YIELD`. Comments in `rt.c` describe rotating within the current priority’s ready list so another same-priority task can run. Yield only matters if the active task is at the front of its ready list (still highest priority overall).

`examples/fair.c` exercises multi-task fairness at one priority.

## Blocking and wake

Blocking ops (sem/mutex/event wait, sleep) move the task off the ready lists into a wait or sleep list and set `enum rt_task_state`. Wake paths mark `RT_TASK_STATE_READY` and reinsert by priority.

Wait lists are **priority-sorted** (`insert_by_priority` uses `priority <` ordering).

## Priority donation interaction

While holding mutexes that have waiters, a task’s **effective** `priority` may be lowered numerically (boosted) toward waiter priorities via `task_donate` / `mutex_donate`. See [Mutexes](Mutexes.md).

## Tick-driven wakeups

`rt_tick_advance()` (from the port’s timer IRQ or signal) processes time; sleeping tasks and timed waits become ready when their wake tick is reached. See [Sleep and Tick](Sleep-and-Tick.md).

## Startup

```c
__attribute__((noreturn)) void rt_start(void);
struct rt_task *rt_start_sched(void);  // first task
```

`rt_start_sched` assumes tasks (including idle) are already on ready lists.

## Not SMP

There is a single `rt_active_task` and one ready bitmap. Stock `rt` is **largely single-hart**; multi-core scheduling is not a documented feature of this tree.
