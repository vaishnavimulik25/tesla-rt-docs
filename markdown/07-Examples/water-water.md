# Example: `water/water.c` (harness)

Supervisor and H₂O task loops shared by every water bonding strategy.

## Shared water scenario (`water.h` / `water.c`)

All water variants implement the same two entry points declared in `water.h`:

```c
void hydrogen(void);
void oxygen(void);
void make_water(void);  /* defined in water.c — increments water_formed */
```

`water/water.c` is **linked into every water Program** (`SConscript`: `env.Program(["water/<variant>.c", water])`). It provides:

| Piece | Role |
|-------|------|
| `make_water()` | `rt_atomic_fetch_add(&water_formed, 1)` |
| `hydrogen_loop` ×2 (prio 2, `2*RT_STACK_MIN`) | Forever: `hydrogen()` then `++hydrogen_bonded` |
| `oxygen_loop` ×1 (prio 1, `2*RT_STACK_MIN`) | Forever: `oxygen()` then `++oxygen_bonded` |
| `timeout` (prio 0) | Sleep `TICKS_TO_RUN` (1000); assert H/O bond counts within one molecule of `water_formed`; `rt_exit` |

Bonding rule under test: **2 H + 1 O → 1 water**. After 1000 ticks, oxygen_bonded ≈ water_formed (±1) and hydrogen_bonded ≈ 2×water (±2) to allow in-flight molecules at exit.

**How variants differ:** only the bodies of `hydrogen` / `oxygen` (barrier vs cond vs sem). The harness and success criteria are identical.


## Tasks / roles

| Task | Priority | Stack | Role |
|------|----------|-------|------|
| `timeout` | 0 | `RT_STACK_MIN` | After 1000 ticks, validate counters, `rt_exit`. |
| `hydrogen_loop` ×2 | 2 | `2 * RT_STACK_MIN` | Call variant `hydrogen()`, count bonds. |
| `oxygen_loop` | 1 | `2 * RT_STACK_MIN` | Call variant `oxygen()`, count bonds. |

Hydrogens outrank oxygen so H threads tend to arrive at the rendezvous first; oxygen still must wait for two H partners inside each strategy.

## Shared state

```c
static rt_atomic_uint32_t hydrogen_bonded, oxygen_bonded, water_formed;
#define TICKS_TO_RUN 1000
```

Plus whatever the linked variant defines in its `struct reaction`.

## Control flow

1. Variant-specific `hydrogen`/`oxygen` rendezvous until `make_water()` runs.
2. Loops bump bond counters **after** the rendezvous returns.
3. Timeout loads `w`, `h`, `o` and asserts:
   - `w-1 ≤ o ≤ w`
   - `2*(w-1) ≤ h ≤ 2*w`
4. `rt_exit` on success.

This file is not built alone; see binary paths under each variant page.

## APIs used

| API | Role |
|-----|------|
| `RT_TASK` | H×2, O×1, timeout. |
| `rt_atomic_*` | Bond / water counters. |
| `rt_assert` | Bounds check at end. |
| `rt_task_sleep` / `rt_exit` | Duration-based pass. |

## Success / failure

- **Pass:** counter bounds hold; `rt_exit`.
- **Fail:** assert messages about too few/many H or O bonds (broken rendezvous / double make_water / deadlock until timeout elsewhere).

## Build / run

Built only as part of a variant, e.g.:

```bash
./build/signal/examples/water/sem
./build/signal/examples/water/barrier
./build/signal/examples/water/cond   # not with cl2000
```

## See also

- [water-sem](water-sem.md) · [water-cond](water-cond.md) · [water-barrier](water-barrier.md)
- [Semaphores](../03-Kernel-Features/Semaphores.md) · [Condition Variables](../03-Kernel-Features/Condition-Variables.md)
- [Building and Examples](../07-Building-and-Examples.md)
