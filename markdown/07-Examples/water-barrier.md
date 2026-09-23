# Example: `water/barrier.c`

Water lab variant: capacity semaphores gate how many H/O enter; a 3-party barrier coordinates “before” and “after” `make_water`.

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

Harness as in [water-water](water-water.md). This file:

- `hydrogen()` — `wait` H-capacity sem (init 2); barrier; barrier; `post` H-capacity.
- `oxygen()` — `wait` O-capacity sem (init 1); barrier; `make_water()`; barrier; `post` O-capacity.

## Shared state (this variant)

```c
struct reaction {
    struct rt_sem h, o;
    struct rt_barrier barrier;
};
static struct reaction rxn = {
    .h = RT_SEM_INIT(rxn.h, 2),
    .o = RT_SEM_INIT(rxn.o, 1),
    .barrier = RT_BARRIER_INIT(rxn.barrier, 3),
};
```

At most 2 hydrogens and 1 oxygen occupy the reaction at once (capacity sems). Barrier count 3 matches one molecule’s participants.

## Control flow

1. Up to 2 H and 1 O pass their capacity semaphores.
2. All three `rt_barrier_wait` — first rendezvous (“everyone present”).
3. Oxygen alone calls `make_water()` between the two barrier waits.
4. Second barrier — “molecule done”; then each posts its capacity sem so the next trio can enter.

**Vs sem/cond:** geometric “all present” sync via barrier; capacity sems prevent extra tasks from joining a half-formed group.

## APIs used

| API | Role |
|-----|------|
| `rt_sem_wait` / `rt_sem_post` | Cap concurrent H (2) and O (1). |
| `rt_barrier_wait` | 3-way phase sync around `make_water`. |
| `RT_BARRIER_INIT` / `RT_SEM_INIT` | Static init inside struct. |

## Success / failure

Harness bounds after 1000 ticks. Barrier mismatch or missing post → deadlock → bond counts too low at timeout.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/water/barrier
```

## See also

- [water-water](water-water.md) · [water-sem](water-sem.md) · [water-cond](water-cond.md)
- [API: barrier](../09-API-Reference/barrier.md) · [Semaphores](../03-Kernel-Features/Semaphores.md)
- [Building and Examples](../07-Building-and-Examples.md)
