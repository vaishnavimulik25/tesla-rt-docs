# C / C++ / Rust

## C (primary)

Public API: `include/rt/*.h`. Link against the arch-specific port objects + `src/*.c`.

Typical includes:

```c
#include <rt/task.h>
#include <rt/mutex.h>
#include <rt/sem.h>
#include <rt/start.h> /* via arch startup */
```

## C++ (`cxx/`)

Header-only style wrappers under `cxx/include/rt/*.hpp`:

`abort`, `barrier`, `cond`, `event`, `exit`, `mutex`, `notify`, `once`, `pool`, `queue`, `rwlock`, `sem`, `task`, `tick`, `timer`, `trap`.

Example: `cxx/include/rt/task.hpp` namespaces thin inline wrappers (`rt::task::yield()`, `sleep`, …).

Examples: `cxx/examples/*.cpp` and `cxx/examples/water/`. Built by SCons into `build/signal/cxx/` on host.

## Rust (`rust/` + crate `rt`)

- Package version **0.20.1**, edition **2024**, `rust-version = "1.97.0"`
- `build = "rust/build.rs"` — **bindgen** + **cc** compile the C kernel for the target
- Lib path: `rust/src/lib.rs`
- Modules: `task`, `sync` (mutex, sem, condvar, event, queue, notify, rwlock, barrier, once, once_lock, lazy_lock, pool), `timer`, `tick`, `cycle`, `stack`, optional `mpu`
- Docs: https://docs.rs/rt

Examples registered in `Cargo.toml` mirror many C demos (`simple`, `donate`, `mutex`, `water-*`, …).

```bash
cargo test
cargo run --example simple
# embedded:
cargo run --target thumbv7em-none-eabihf --features qemu-m7 --example simple
```

## Choosing a language

| Need | Prefer |
|------|--------|
| Bare-metal bring-up / ISR stubs | C |
| RAII guards without writing cleanup macros | C++ wrappers or Rust |
| `no_std` app crate integrating `rt` | Rust crate |
| Exact ABI / assembly ports | C headers + arch asm |
