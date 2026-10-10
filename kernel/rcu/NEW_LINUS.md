# The New Linus: `kernel/rcu/`

Week 6 (Nov 16-22): concurrency and interrupts. Lab:
[`labs/06-locking`](../../labs/06-locking/NEW_LINUS.md)

## What lives here

RCU, Read-Copy-Update: the synchronisation mechanism that lets readers run with
no locks and no atomic writes at all, which is why routing lookups, the
`task_struct` list and most networking fast paths scale across hundreds of
CPUs. Writers publish a new copy of the data, then wait until every reader that
could still see the old copy has finished (a **grace period**) before freeing it.

Read `Documentation/RCU/whatisRCU.rst` sections 1-3 before any code. It is one
of the best documents in the tree.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `include/linux/rcupdate.h` | `rcu_read_lock()`, `rcu_read_unlock()` | On a non-preemptible kernel these compile to almost nothing. That is the point. |
| `include/linux/rcupdate.h` | `rcu_dereference()`, `rcu_assign_pointer()` | Publish and subscribe: the memory-ordering guarantees that make a half-initialised object impossible to see. |
| `include/linux/rculist.h` | `list_add_rcu()`, `list_for_each_entry_rcu()` | RCU versions of the list API you used in week 2. |
| `tree.c` | `synchronize_rcu()`, `call_rcu()` | Blocking wait for a grace period vs. a callback after it. |
| `tree.c` | `rcu_gp_kthread()` | The kernel thread that drives grace periods. Skim only. |

## Watch it in the VM

```sh
ps -e -o pid,comm | grep -i rcu            # rcu_preempt / rcu_sched, rcuog, ...
grep RCU /proc/softirqs                     # RCU callbacks run as a softirq
```

## Words to own

- **RCU**: readers proceed without locks; writers copy, publish, then free after
  a grace period.
- **read-side critical section**: code between `rcu_read_lock()` and
  `rcu_read_unlock()`.
- **grace period**: interval after which every pre-existing reader has finished.
- **quiescent state**: a point where a CPU holds no RCU references (for
  example, a context switch).
- **publish/subscribe**: `rcu_assign_pointer()` / `rcu_dereference()`.
- **memory barrier**: an instruction that orders memory accesses
  (`Documentation/memory-barriers.txt`).

## Answer in your notes

1. Why is a context switch a quiescent state?
2. Why can't you call `synchronize_rcu()` while holding a spinlock?
3. In lab 06, convert the reader of your list to RCU. What does the writer have
   to do differently?
