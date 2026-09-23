# Other Synchronization Primitives

This page covers barriers, reader-writer locks, once-calls, object pools, and related helpers. Mutexes, semaphores, condition variables, events, queues, and notifications have dedicated pages.

---

## Barriers (`include/rt/barrier.h`)

```c
bool rt_barrier_wait(struct rt_barrier *barrier);

RT_BARRIER(name, count);
struct rt_barrier b = RT_BARRIER_INIT(b, count);
```

**Purpose:** Block until `count` tasks have called `rt_barrier_wait`. Exactly **one** caller in each generation receives `true` (the “serial” thread); others receive `false`. The barrier then resets for the next generation.

**Implementation:** Mutex + condition variable + generation counter (`level`, `threshold`, `generation`).

**ISR:** Not for ISRs (blocks on mutex/cond).

**Example:** `examples/water/barrier.c`.

**Pitfalls:** Mismatched `count` vs number of callers deadlocks; the `true` return is only for one task per generation.

---

## Reader-writer locks (`include/rt/rwlock.h`)

```c
void rt_rwlock_rdlock(struct rt_rwlock *lock);
bool rt_rwlock_tryrdlock(struct rt_rwlock *lock);
bool rt_rwlock_timedrdlock(struct rt_rwlock *lock, unsigned long ticks);
void rt_rwlock_rdunlock(struct rt_rwlock *lock);

void rt_rwlock_wrlock(struct rt_rwlock *lock);
bool rt_rwlock_trywrlock(struct rt_rwlock *lock);
bool rt_rwlock_timedwrlock(struct rt_rwlock *lock, unsigned long ticks);
void rt_rwlock_wrunlock(struct rt_rwlock *lock);

void rt_rwlock_unlock(struct rt_rwlock *lock);  /* unlock either mode */

RT_RWLOCK(name);
RT_RWLOCK_READ_GUARD(guard, lock);
RT_RWLOCK_WRITE_GUARD(guard, lock);
/* + _ADOPT variants */
```

**Purpose:** Concurrent readers **or** a single writer.

**Implementation:** Atomic reader count + mutex + cond.

**ISR:** Blocking locks are task-only; try/timed(0) may be used carefully.

**Example:** `examples/rwlock.c`.

**Pitfalls:** Writer starvation possible under continuous readers (check policy in `src/rwlock.c` for your revision); always unlock the same mode you locked (or use `rt_rwlock_unlock`).

---

## Once (`include/rt/once.h`)

```c
void rt_once_call(struct rt_once *once, void (*fn)(void));

RT_ONCE(name);
struct rt_once o = RT_ONCE_INIT(o);
```

**Purpose:** Exactly one caller runs `fn`; all callers block until `fn` returns (initialization rendezvous).

**Implementation:** Atomic `done` flag + mutex.

**ISR:** No (may lock mutex / run `fn`).

**Example:** `examples/once.c` (also shows resetting `done` for test stress — **not** a supported public reset API).

---

## Object pool (`include/rt/pool.h`)

```c
void *rt_pool_get(struct rt_pool *pool);  /* blocks until an object is free */
void rt_pool_put(struct rt_pool *pool, void *obj);

RT_POOL(name, objects_array);
RT_POOL_STATIC(name, objects_array);
```

**Purpose:** Fixed array of objects handed out / returned with blocking get.

**Implementation:** Mutex + cond + pointer table filled by a `constructor` that points at each array element.

**ISR:** No for blocking `get`.

**Example:** `examples/pool.c`.

---

## Priority donation (mutex-related)

Donation is part of [Mutexes](Mutexes.md), not a separate object. Example: `examples/donate.c`.

There is **no** standalone `rt_donate_*` user API — donation is triggered by waiting on a mutex held by a lower-priority task.

---

## Lists and containers (mostly internal)

`include/rt/list.h` — intrusive doubly linked lists used by the kernel (`rt_list_push_back`, `rt_list_insert_by`, …).

`include/rt/container.h` — `container_of`-style helpers for kernel structures.

Application code rarely needs these unless extending the kernel.

---

## Pendable functions (`include/rt/pend_function.h`)

```c
bool rt_pend_function(struct rt_syscall_record *record,
                      void (*fn)(uintptr_t), uintptr_t arg);
RT_PEND_FUNCTION_RECORD(name);
```

Request `fn(arg)` to run in the **pending syscall handler** (useful from ISRs). Returns `false` if the record is already pending. See API reference for architecture caveats.
