# Example: `water/sem.c`

Water lab variant: pair hydrogens with an atomic counter + `h2ready` semaphore; oxygen waits, makes water, posts `hdone` twice.

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

Harness tasks from [water-water](water-water.md). This file only implements:

- `hydrogen()` — every second H (odd fetch_add) posts `h2ready`; every H waits on `hdone`.
- `oxygen()` — wait `h2ready`, `make_water()`, `rt_sem_post_n(&hdone, 2)`.

## Shared state (this variant)

```c
struct reaction {
    struct rt_sem h2ready, hdone;
    rt_atomic_uint h;
};
static struct reaction rxn = {
    .h2ready = RT_SEM_INIT(rxn.h2ready, 0),
    .hdone   = RT_SEM_INIT(rxn.hdone, 0),
    .h = 0,
};
```

## Control flow

1. Each hydrogen does `fetch_add(&h, 1)`. If the **old** value was odd (`& 1 == 1`), this H is the second of a pair → `rt_sem_post(&h2ready)`.
2. Every hydrogen then blocks on `hdone` until oxygen finishes the molecule.
3. Oxygen waits for a ready H₂ pair, calls `make_water()`, then `rt_sem_post_n(&hdone, 2)` to release both hydrogens.

**Vs other variants:** no mutex/cond predicate; pairing is lock-free counting plus semaphores. No barrier — release is explicit `post_n`.

## APIs used

| API | Role in this variant |
|-----|----------------------|
| `rt_atomic_fetch_add` | Detect every 2nd hydrogen. |
| `rt_sem_wait` / `rt_sem_post` / `rt_sem_post_n` | H₂ ready + done handshake. |
| `make_water` | Count a formed molecule. |

## Success / failure

Same as harness: after 1000 ticks, H/O counters match water (±1 molecule). Deadlock → timeout never sees enough bonds → assert fail (or hang if timeout also stuck — here timeout does not take the sync objects).

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/water/sem
```

## See also

- [water-water](water-water.md) · [water-cond](water-cond.md) · [water-barrier](water-barrier.md)
- [Semaphores](../03-Kernel-Features/Semaphores.md) · [sem](sem.md)
- [Building and Examples](../07-Building-and-Examples.md)
