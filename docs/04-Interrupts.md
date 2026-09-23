# Interrupts

## Detection

```c
#include <rt/interrupt.h>

bool rt_interrupt_is_active(void);  // true if called from an interrupt
```

Generic APIs such as `rt_sem_post` / `rt_event_set` consult this to choose a synchronous syscall vs a **pendable** path.

## Design rules (from README + headers)

1. **Non-blocking interfaces** may be used from tasks **and** interrupts.
2. Synchronization does **not** rely on disabling IRQs on targets with hardware atomics; IRQ masking is reserved mainly for atomic emulation on weak cores (e.g. Armv6-M).
3. Work that must wake blocked tasks from ISR context typically **posts** (atomically) and then **pends** a deferred syscall (`rt_syscall_pend`) so the scheduler runs after other interrupts, before the next task.

## Pendable syscalls (ISR-oriented)

From `rt/syscall.h`, `enum rt_syscall_pendable`:

| Pendable | Role |
|----------|------|
| `RT_SYSCALL_PENDABLE_SEM_POST` | Complete semaphore wake after ISR post |
| `RT_SYSCALL_PENDABLE_EVENT_SET` | Complete event wake after ISR set |
| `RT_SYSCALL_PENDABLE_TICK` | Tick processing |
| `RT_SYSCALL_PENDABLE_FUNCTION` | Run a deferred function (`rt_pend_function`) |

Architecture ports map pend to PendSV (Cortex-M), a lowest-priority soft IRQ (A/R), machine software interrupt (RISC-V), or a POSIX signal (`arch/signal`).

## Explicit ISR / task post APIs

When the call site is known, prefer the specialized helpers:

**Semaphores** (`rt/sem.h`):

- `rt_sem_post_from_task` / `rt_sem_post_n_from_task`
- `rt_sem_post_from_interrupt` / `rt_sem_post_n_from_interrupt`

**Events** (`rt/event.h`):

- `rt_event_set_from_task`
- `rt_event_set_from_interrupt`

Generic `rt_sem_post` / `rt_event_set` auto-select based on `rt_interrupt_is_active()`.

## Queues from interrupts

`rt/queue.h` documents a bounded MPMC lock-free queue supporting blocking, timed, and non-blocking push/pop/peek, with as many concurrent accessors (threads/interrupts) as there are slots. Use **try** / **timed** variants from ISRs so you never block in interrupt context.

## Deferred work: `rt_pend_function`

```c
#include <rt/pend_function.h>

RT_PEND_FUNCTION_RECORD(my_record);

bool rt_pend_function(struct rt_syscall_record *record,
                      void (*fn)(uintptr_t), uintptr_t arg);
```

Header notes:

- Callable from an **interrupt** or a **privileged** task (needs privileged data structures).
- Unprivileged tasks may fault.
- On some architectures (notably Arm A/R), a task-triggered pend may not run immediately; do not issue a synchronous syscall before the pendable handler has run unless the platform guarantees ordering.

## Port-specific IRQ setup (summary)

| Port | Tick | Sync syscall | Pendable |
|------|------|--------------|----------|
| Arm M | SysTick → `rt_tick_advance` (or custom timer) | `rt_svcall_handler` | `rt_pendsv_handler` (lowest prio) |
| Arm A/R | Timer IRQ → `rt_tick_advance` | `rt_svc_handler` | `rt_syscall_irq_handler` (lowest IRQ) |
| RISC-V | Timer + MSI | `rt_ecall_handler` via trap vector | `rt_msi_handler` (`MSIP_BASE`) |
| signal | SIGALRM-style tick | assembly sync path | signal handler |

See arch READMEs under `arch/*/README.md` and [Porting Guide](10-Porting-Guide.md).

## What not to do in ISRs

- Do not call blocking waits (`rt_sem_wait`, `rt_mutex_lock`, `rt_task_sleep`, …).
- Do not assume IRQ-disable critical sections are required for `rt` atomics on LDREX/STREX-class cores.
- Prefer try/post/set + pend over long ISR work; use `rt_pend_function` when mutual exclusion with other syscalls is required.
