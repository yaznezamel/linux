# Lab 06: locking

Week 6 (Nov 16-22). Read first:
[`kernel/locking/NEW_LINUS.md`](../../kernel/locking/NEW_LINUS.md),
[`kernel/rcu/NEW_LINUS.md`](../../kernel/rcu/NEW_LINUS.md),
[`kernel/irq/NEW_LINUS.md`](../../kernel/irq/NEW_LINUS.md)

## Goal

Cause a real race, fix it three ways, and make lockdep catch a deadlock that
hasn't happened yet.

## You write

- `race.c` (user space, C review)
- `race_mod.c`, `abba.c`, `Makefile` (modules)

## Steps

1. Boot with four CPUs: `vng -p 4 --user root`.
2. `race_mod.c`: on load, start 4 kernel threads with `kthread_run()`. Each
   adds 1 to a shared `unsigned long counter` one million times, then waits
   for `kthread_should_stop()`. On unload, `kthread_stop()` them and print the
   counter. Expect less than 4,000,000.
3. Fix it with a `spinlock_t`. Then with `atomic_long_t`. Time each version
   (`ktime_get()` before and after) and print the cost.
4. `abba.c`: two spinlocks A and B. In init, take A then B, release; then take
   B then A, release. It never deadlocks (one thread), but lockdep reports a
   "possible circular locking dependency". Save the report.
5. Stretch: go back to lab 02's list module and make the reader use
   `rcu_read_lock()` + `list_for_each_entry_rcu()` and the deleter use
   `list_del_rcu()` + `kfree_rcu()` (your struct needs a `struct rcu_head` member for it).

## Done when

- [ ] Unlocked counter is wrong, both fixed versions print exactly 4,000,000.
- [ ] You have numbers for the cost of the spinlock vs the atomic.
- [ ] The lockdep report is in your notes with every line explained.

## Hints

- `include/linux/kthread.h`, `include/linux/spinlock.h`, `include/linux/atomic.h`
- `include/linux/timekeeping.h`: `ktime_get()`; `include/linux/ktime.h`: `ktime_to_ns()`
- A kthread function must not return before `kthread_stop()` is called, or
  `kthread_stop()` will act on a freed task.
- If the unlocked counter always comes out right, the compiler has turned your
  loop into one big addition. Write the increment as
  `WRITE_ONCE(counter, READ_ONCE(counter) + 1);` so each step is a real load
  and store. (In `race.c`, use a `volatile` counter for the same reason.)

## C review: threads and atomics in user space

`race.c`: the same race with 4 `pthread`s; fix it with
`pthread_mutex_t`, then with `<stdatomic.h>` `atomic_fetch_add()`.
Compile with `gcc -O2 -pthread`.
