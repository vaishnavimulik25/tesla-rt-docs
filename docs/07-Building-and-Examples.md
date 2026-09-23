# Building and Examples

This page explains how to build the open-source `rt` kernel (RTNG) and how every example under `examples/` behaves. For board-specific BSPs, see [Architecture and Ports](06-Architecture-and-Ports.md) and [Supported Devices](Supported-Devices.md).

## Host build (recommended first step)

From a clone of [https://git.rtng.org/rt/rt](https://git.rtng.org/rt/rt):

```bash
# Host / POSIX signal simulation (arch/signal)
scons -j$(nproc)

# Optional: Address/UB sanitizers, debug-friendly opts, tracing
scons --sanitize
scons --debug-optimization
scons --trace
```

Artifacts land under `build/signal/` (and related variant dirs). The root `SConstruct` builds `librt`, the `arch/signal` port, C and C++ examples, and enables `RT_CYCLE_ENABLE` / `RT_TASK_CYCLE_ENABLE` for host cycle benchmarks.

### Run host examples

```bash
./build/signal/examples/simple
./build/signal/examples/mutex
# …each Program() target from examples/SConscript
```

Exact binary names follow SCons `Program(...)` paths (e.g. `cycle/mutex`, `water/sem`).

### `test.bash` / QEMU

```bash
./test.bash              # host signal + Cargo checks as scripted
./test.bash m7           # QEMU Cortex-M7
./test.bash m55|r52|a53|rv32|rv64
```

QEMU glue lives under `qemu/`. Cargo features include `qemu-m7`, `qemu-m55`, `qemu-r52`, `qemu-a53`, `qemu-rv32`, `qemu-rv64`, plus `task-mpu` where applicable. See `test.bash` for machine flags and timeouts.

### Docker

`Dockerfile` + `docker.bash` provide a containerized toolchain environment.

### Toolchain notes

- Host: clang/clang++, C17, GNU++17, `-fno-exceptions -fno-rtti`.
- Examples that use `__attribute__((cleanup))` guards (`mutex`, `cond`, `fair`, `recursive`, `rwlock`, `cycle/mutex`, `water/cond`) are **skipped** when `cl2000` is the compiler (`examples/SConscript`).

## Example catalog

Each example has a full source walkthrough (tasks/roles, shared state, control flow, APIs, pass/fail, build) under [07-Examples/](07-Examples/index.md). Summary table below; prefer the per-example pages for behavior detail.

| Path | Demonstrates | Key APIs |
|------|----------------|----------|
| `simple.c` | Two arg-tasks yielding then exiting | `RT_TASK_ARG`, `rt_task_yield`, `rt_task_drop_privilege`, `rt_exit` |
| `empty.c` | Minimal single task | `RT_TASK`, `rt_exit` |
| `sem.c` | Counting semaphore post/wait | `RT_SEM`, `rt_sem_post`, `rt_sem_wait` |
| `mutex.c` | lock / trylock / timedlock + guards | `RT_MUTEX`, `RT_MUTEX_GUARD`, `rt_mutex_*` |
| `recursive.c` | Recursive mutex | `RT_MUTEX_RECURSIVE` |
| `donate.c` | Priority donation under inversion | nested `rt_mutex_lock`, priorities 0…3 |
| `fair.c` | Equal-priority mutex fairness | `RT_MUTEX_GUARD`, exit via `rt_sem_trywait` |
| `cond.c` | Condition variable + mutex | `RT_COND`, `rt_cond_signal/wait` |
| `event.c` | Event bits ANY/ALL/NOCLEAR | `RT_EVENT`, `rt_event_set/timedwait` |
| `notify.c` | Task notification value | `RT_NOTIFY`, `rt_notify_post/wait` |
| `queue.c` | MPMC queue | `RT_QUEUE_STATIC`, `rt_queue_push/pop` |
| `rwlock.c` | Reader/writer lock | `RT_RWLOCK`, `rt_rwlock_rdlock/wrlock` |
| `once.c` | One-time init | `RT_ONCE`, `rt_once_call` |
| `pool.c` | Blocking object pool | `RT_POOL_STATIC`, `rt_pool_get/put` |
| `timer.c` | Software timer daemon | `RT_TIMER_*`, `rt_timer_start/stop` |
| `sleep.c` | Periodic sleep accuracy | `rt_task_sleep_periodic`, `rt_tick_count` |
| `tls.c` | Task-local storage | `rt_tls_*` / `_Thread_local` when enabled |
| `float.c` | FP context save/restore | float ops across yields |
| `water/*` | Classic “make water” sync lab | barrier/cond/sem variants |

### Cycle micro-benchmarks (`examples/cycle/`)

Each records `rt_cycle()` deltas for N iterations, then calls `rt_trap()`:

| File | Measures |
|------|----------|
| `yield.c` | Yield-to-peer latency (equal priority) |
| `sleep.c` | Sleep wake latency |
| `sem.c` | post→wait |
| `mutex.c` | contended mutex handoff |
| `event.c` | set→wait |
| `notify.c` | post→wait |
| `queue.c` | push→pop |

These are **instrumentation harnesses**, not published scorecards. See [Benchmarks](12-Benchmarks.md).

### Water lab (`examples/water/`)

Shared `water.c` / `water.h` spawn H/O tasks and a timeout supervisor. Bonding strategies:

| File | Primitive |
|------|-----------|
| `water/sem.c` | Semaphores + atomics |
| `water/cond.c` | Mutex + condition variables |
| `water/barrier.c` | Semaphores + barrier |

## Board / BSP examples

External repos under [https://git.rtng.org/rt](https://git.rtng.org/rt) add `examples/` such as `blinky`, IRQ/RTI tests, etc. Build with that repo’s `SConstruct` and board/MCU config. See [Supported Devices](Supported-Devices.md).
