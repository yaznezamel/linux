# The New Linus: `kernel/locking/`

Week 6 (Nov 16-22): concurrency and interrupts. Lab:
[`labs/06-locking`](../../labs/06-locking/NEW_LINUS.md). Same week:
[`kernel/rcu/`](../rcu/NEW_LINUS.md), [`kernel/irq/`](../irq/NEW_LINUS.md).

## What lives here

Every lock the kernel uses, and lockdep, the runtime checker that proves your
locking order can't deadlock. The one rule behind most of this folder: **in
atomic context you may not sleep**. Spinlocks are for atomic context (they
busy-wait and disable preemption); mutexes may sleep and are for process
context only.

Read `Documentation/locking/locktypes.rst` first, then `spinlocks.rst`.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `include/linux/spinlock.h` | `spin_lock()`, `spin_lock_irqsave()`, `spin_lock_bh()` | Which variant to use depends on who else can take the lock: another CPU, an interrupt, or a softirq. |
| `spinlock.c` | `_raw_spin_lock()` | Disable preemption, then take the lock. |
| `include/asm-generic/qspinlock.h` | `queued_spin_lock()` | The fast path: one atomic compare-and-swap when the lock is free. |
| `qspinlock.c` | `queued_spin_lock_slowpath()` | Under contention, waiters queue (MCS lock) so each spins on its own cache line. |
| `mutex.c` | `mutex_lock()`, `__mutex_lock_slowpath()`, `__mutex_lock_common()` | Fast path is one atomic op; slow path spins briefly if the owner is running (optimistic spinning), then sleeps. |
| `lockdep.c` | `check_prev_add()`, `check_deadlock()` | lockdep records every "lock B taken while holding A" pair and reports a cycle the first time it becomes possible, before it deadlocks. |

## Watch it in the VM

Needs `CONFIG_PROVE_LOCKING=y` (part of the week 1 learning config):

```sh
cat /proc/lockdep_stats | head            # lock classes and dependencies seen
dmesg | grep -i 'lockdep\|possible circular'   # after loading lab 06's bad module
```

## Words to own

- **race condition**: a result that depends on timing between CPUs or contexts.
- **critical section**: code that must not run concurrently with itself.
- **spinlock**: busy-waiting lock; holder must not sleep.
- **mutex**: sleeping lock with an owner; process context only.
- **atomic context**: interrupt, softirq, or holding a spinlock; sleeping is a bug.
- **preemption**: being switched out involuntarily; a spinlock disables it.
- **contention**: several CPUs wanting the same lock at once.
- **MCS lock**: queue lock where each waiter spins on its own variable.
- **lockdep**: lock dependency validator.
- **AB-BA deadlock**: two paths taking the same two locks in opposite order.

## Answer in your notes

1. Data is shared between process context and a hardirq handler. Which
   `spin_lock*()` variant must the process-context side use, and why?
2. Why does a mutex spin before sleeping?
3. Paste the lockdep report from lab 06 into your notes and explain each line
   of the "possible circular locking dependency" chain.
