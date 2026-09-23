# Getting Started

This walkthrough uses the **host POSIX simulation** port `arch/signal` — useful for learning and CI. It is **not** a chip port (see [Architecture and Ports](06-Architecture-and-Ports.md)).

## Prerequisites

From the tree’s build files (`SConstruct`, `test.bash`):

- **clang** / **clang++** (host builds use LLVM flags)
- **SCons**
- Optional: **Rust** toolchain (edition 2024 / rust-version 1.97.0 per `Cargo.toml`) for crate examples
- Optional QEMU + cross toolchains for embedded targets (`test.bash`)

## Clone

```bash
git clone https://git.rtng.org/rt/rt
cd rt
# Local clone used for these docs: /workspace/rtng-rt
```

## Host build (SCons)

From the repository root:

```bash
scons
```

Outputs land under `build/`:

- `build/lib/` — kernel library objects  
- `build/signal/` — host example binaries (and maps)

Useful SCons options (from `SConstruct`):

| Option | Effect |
|--------|--------|
| `--sanitize` | Address + undefined sanitizers |
| `--debug-optimization` | `-Og` instead of `-O3 -flto` |
| `--trace` | `-DRT_TRACE_ENABLE=1` |
| `--log` | `-DRT_LOG_ENABLE=1` |
| `--clang-tidy` | Run clang-tidy after each compile |

Host builds define `RT_CYCLE_ENABLE=1` and `RT_TASK_CYCLE_ENABLE=1` by default in `SConstruct`.

## Run a simple example

`examples/simple.c` creates two same-priority tasks that yield in a loop; one then calls `rt_exit()`:

```c
#include <rt/exit.h>
#include <rt/task.h>

static void simple(uintptr_t arg)
{
    rt_task_drop_privilege();
    for (int i = 0; i < 100; ++i)
    {
        rt_task_yield();
    }

    if (arg == 1)
    {
        rt_exit();
    }
}

RT_TASK_ARG(simple, 0, RT_STACK_MIN, 0);
RT_TASK_ARG(simple, 1, RT_STACK_MIN, 0);
```

After `scons`:

```bash
./build/signal/simple
# (or timeout wrapper as in test.bash)
```

### What happens

1. Each `RT_TASK_ARG` expands to a **constructor** that allocates a static stack (`RT_STACK`), initializes `struct rt_task`, builds a context via `rt_context_init`, and calls `rt_task_init` (allowed only before `rt_start`).
2. Architecture startup eventually calls `rt_start()` → `rt_start_sched()`, which picks the highest-priority ready task (numerically smallest priority).
3. Tasks call `rt_task_yield()` (syscall `RT_SYSCALL_YIELD`) to round-robin with peers.
4. `rt_exit()` ends the simulation on the signal port (architecture-specific).

## Quick Rust / Cargo path

```bash
cargo test          # host
./test.bash         # host examples + clippy (see script)
./test.bash m7      # QEMU Cortex-M7 example matrix
```

## Next examples to try

| Example | Demonstrates |
|---------|----------------|
| `examples/sem.c` | Timed semaphore wait/post |
| `examples/mutex.c` | Mutex + `RT_MUTEX_GUARD` |
| `examples/donate.c` | Priority donation |
| `examples/queue.c` | Bounded MPMC queue |
| `examples/timer.c` | Software timer daemon |

More detail: [Building and Examples](07-Building-and-Examples.md).
