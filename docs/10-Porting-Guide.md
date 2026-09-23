# Porting Guide

High-level guide distilled from `arch/*/README.md`, `rt/syscall.h`, `rt/context.h`, and port sources. Not a line-by-line BSP cookbook.

## What a port must provide

1. **Context switch** — `rt_context_init(fn, arg, stack, stack_size)` and switch machinery using `rt_context_prev` / task `ctx` field (`RT_TASK_CTX_OFFSET`).
2. **Synchronous syscalls** — `rt_syscall_0/1/2/3` that run `rt_syscall_run` before returning (SVC / ecall / trap / signal).
3. **Pendable syscalls** — `rt_syscall_pend` + handler invoking `rt_syscall_run_pending`, at **lowest** interrupt priority so it cannot preempt other IRQs incorrectly.
4. **Tick** — periodic call to `rt_tick_advance()`.
5. **Interrupt active flag** — `rt_interrupt_is_active()`.
6. **Stack attributes** — `rt/arch/stack.h` (`RT_STACK_ALIGN`, `RT_STACK_MIN`, …).
7. **Startup** — after BSS/ctors, call `rt_start()` (often from asm).
8. Optional: TLS (`rt_tls_set`/`get`), MPU, cycle counter, abort/exit/trap/panic backends, semihosting.

## Arm Cortex-M checklist

From `arch/arm/README.md`:

- [ ] Vector `SVCall` → `rt_svcall_handler`
- [ ] Vector `PendSV` → `rt_pendsv_handler` (lowest prio)
- [ ] SysTick (or timer) → `rt_tick_advance` (prio ≥ PendSV numerically ≤)
- [ ] Tasks on PSP; consider PSPLIM on v8-M
- [ ] Decide MPU / privilege policy

## Arm Cortex-A/R checklist

- [ ] Include the right `arch/arm/ar/...` interrupt controller support on the include path
- [ ] SVC → `rt_svc_handler`; soft IRQ → `rt_syscall_irq_handler` (lowest IRQ)
- [ ] Timer IRQ calls `rt_tick_advance`
- [ ] Use `RT_IRQ_HANDLER_*` macros; clear interrupt sources before return
- [ ] Lazy FP via UDF handler macros if needed

## RISC-V checklist

From `arch/riscv/README.md`:

- [ ] ISA: A + Zicsr (M/Zbb recommended)
- [ ] `rt_trap_vector` ecall slots → `rt_ecall_handler`
- [ ] Top-level sync trap → `rt_trap_handler`
- [ ] Define `MSIP_BASE`; route MSI → `rt_msi_handler`
- [ ] Reservation clear behavior on exception entry — verify for your core

## Host `signal` port

Use as a reference for syscall/tick **semantics** without hardware: signals emulate async entry; one thread switches stacks. Do not treat it as a template for MCU exception priorities.

## AArch64 / C28

Follow existing `arch/aarch64` / `arch/c28` sources and QEMU/`SConscript` patterns. Narrative port docs beyond the code are limited (**not documented in-tree** as standalone READMEs for these two).

## Validation

1. `examples/simple` and `examples/sem` on the new port  
2. `examples/donate` if mutex donation matters  
3. An ISR that `rt_sem_post_from_interrupt` / `rt_event_set_from_interrupt`  
4. Optional: `examples/cycle/*` once `rt_cycle` works  
