# The New Linus: `kernel/sched/`

Week 4 (Nov 2-8): the scheduler. Lab:
[`labs/04-sched`](../../labs/04-sched/NEW_LINUS.md)

## What lives here

The code that decides which task runs on which CPU, and for how long. It is
split into scheduling classes, tried in priority order: `stop`, `dl`
(deadline), `rt` (real-time FIFO/RR), `fair` (normal tasks), `ext` (BPF
schedulers), `idle`. Normal processes use `fair`, which has run the EEVDF
algorithm since Linux 6.6, replacing CFS.

Read `Documentation/scheduler/sched-eevdf.rst` before the code.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `sched.h` | `struct rq`, `struct sched_class` | One runqueue per CPU; each class is an ops table (`enqueue_task`, `pick_next_task`, ...). |
| `core.c` | `schedule()` → `__schedule()` | Pick the next task and switch to it. Find the call to `pick_next_task()` and `context_switch()`. |
| `core.c` | `context_switch()` | Switch address space (`mm`) and then registers and stack. |
| `core.c` | `try_to_wake_up()` | How a sleeping task becomes runnable and which CPU it lands on. |
| `core.c` | `sched_tick()` | Called from the timer interrupt; this is where preemption of a running task is decided. |
| `fair.c` | `update_curr()` | Charges CPU time to the running task as weighted virtual runtime. |
| `fair.c` | `entity_eligible()`, `pick_eevdf()` | EEVDF itself: among tasks that are owed CPU time (eligible), pick the earliest virtual deadline. |
| `fair.c` | `place_entity()` | Where a waking task is placed, using its saved lag. |
| `include/linux/sched.h` | `struct sched_entity` | Fields `vruntime`, `deadline`, `vlag`, `slice`. |
| `ext/ext.c` | top-of-file comment | sched_ext: a scheduling policy written as a BPF program and loaded at runtime. |

## Watch it in the VM

```sh
vng -p 2                                  # boot with 2 CPUs
cat /proc/self/sched | head -20           # se.vruntime, nr_switches, ...
nice -n 10 sh -c 'while :; do :; done' &
cat /proc/$!/sched | grep -E 'vruntime|nr_switches|prio'
chrt -p $$                                # this shell's policy and priority

cd /sys/kernel/tracing
echo 1 > events/sched/sched_switch/enable
sleep 1; echo 0 > tracing_on
grep -v '^#' trace | head                 # prev_comm=... ==> next_comm=...
```

## Words to own

- **runqueue**: per-CPU structure holding runnable tasks (`struct rq`).
- **scheduling class**: a pluggable policy implemented as a `struct sched_class`.
- **EEVDF**: Earliest Eligible Virtual Deadline First, the fair-class
  algorithm since 6.6.
- **vruntime**: CPU time scaled by the task's weight (from its nice value).
- **lag**: service a task is owed (positive) or has overdrawn (negative);
  eligible means lag >= 0.
- **preemption**: taking the CPU from a running task without it yielding.
- **context switch**: saving one task's CPU state and loading another's.
- **sched_ext**: BPF-defined scheduler classes, merged in 6.12.

## Answer in your notes

1. Two CPU-bound loops on one CPU, nice 0 and nice 10: what CPU share does each
   get? Measure it in lab 04 and explain it with weights from
   `sched_prio_to_weight[]` in `core.c`.
2. What does `context_switch()` skip when switching between two threads of the
   same process?
3. In one sentence each: what did CFS optimise for, and what does EEVDF add?
