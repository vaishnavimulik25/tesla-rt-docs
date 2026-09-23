# Example: `water/cond.c`

Water lab variant: mutex + two condition variables track available / bonded hydrogens.

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

- `hydrogen()` — under mutex: `++havailable`, signal `hready`; wait until `hbonded > 0`, then `--hbonded`.
- `oxygen()` — under mutex: wait until `havailable >= 2`, `make_water()`, consume 2 available, add 2 bonded; after unlock, signal `hdone` twice.

## Shared state (this variant)

```c
struct reaction {
    int havailable, hbonded;
    struct rt_cond hready, hdone;
    struct rt_mutex m;
};
```

## Control flow

1. Hydrogens announce availability on `hready`.
2. Oxygen waits for `havailable >= 2` (predicate loop + `rt_cond_wait`).
3. Oxygen forms water, moves 2 H from available → bonded, drops the mutex, then signals `hdone` twice so both waiting hydrogens can decrement `hbonded` and leave.

Comment in source: by the time oxygen signals, ≥2 hydrogens are in `cond_wait` on `hdone`, so neither signal is lost.

**Vs sem variant:** explicit predicates and mutex; easier to read, uses condvar API. **Vs barrier:** no 3-way barrier phases — bonding is counted in shared ints.

> Skipped for `cl2000` (cleanup / guards).

## APIs used

| API | Role |
|-----|------|
| `RT_MUTEX_GUARD` / mutex | Protect availability counters. |
| `rt_cond_wait` / `rt_cond_signal` | H ready / H done rendezvous. |
| `make_water` | Molecule count. |

## Success / failure

Harness bounds on bond counters after 1000 ticks.

## Build / run

```bash
scons -j$(nproc)
./build/signal/examples/water/cond
```

## See also

- [water-water](water-water.md) · [water-sem](water-sem.md) · [water-barrier](water-barrier.md)
- [Condition Variables](../03-Kernel-Features/Condition-Variables.md) · [cond](cond.md)
- [Building and Examples](../07-Building-and-Examples.md)
