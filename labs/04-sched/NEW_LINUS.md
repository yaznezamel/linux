# Lab 04: sched

Week 4 (Nov 2-8). Read first: [`kernel/sched/NEW_LINUS.md`](../../kernel/sched/NEW_LINUS.md)

## Goal

Measure the scheduler instead of reading about it: predict the CPU share of two
competing processes from the kernel's weight table, then check the prediction.

## You write

- `spin.c` (user space)
- `dispatch.c` (user space, C review)
- `RESULTS.md`: your numbers and the explanation

## Steps

1. `spin.c`: loop for N seconds (argument) counting iterations; print
   iterations per second and the PID at the end.
2. Boot with two CPUs: `vng -p 2 --user root`.
3. Pin two copies to the same CPU, one at nice 0 and one at nice 10:
   ```sh
   taskset -c 0 ./spin 10 &
   taskset -c 0 nice -n 10 ./spin 10 &
   wait
   ```
4. Look up the weights for nice 0 and nice 10 in `sched_prio_to_weight[]`
   (`kernel/sched/core.c`). Predict each one's share: `w / (w0 + w10)`. Compare
   with your measured iteration rates.
5. While they run, read `/proc/PID/sched` for both and compare `se.vruntime`
   and `nr_involuntary_switches`.
6. Repeat with one copy as real-time: `chrt -f 10 ./spin 10`. What happens to
   the fair-class task, and why does it still get a little CPU? (Look up
   `sched_rt_runtime_us` in `/proc/sys/kernel/`.)

## Done when

- [ ] `RESULTS.md` has predicted vs measured share, within a few percent.
- [ ] You can explain the real-time result using `sched_rt_runtime_us`.

## C review: function pointers (K&R 5.11)

`dispatch.c`: define `struct policy { const char *name; int (*pick)(int *tasks, int n); };`
with two policies (pick the first, pick the smallest) in an array, and call
each through the pointer. That is a `struct sched_class` in miniature.
