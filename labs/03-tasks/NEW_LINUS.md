# Lab 03: tasks

Week 3 (Oct 26-Nov 1). Read first: [`kernel/NEW_LINUS.md`](../../kernel/NEW_LINUS.md),
[`arch/x86/entry/NEW_LINUS.md`](../../arch/x86/entry/NEW_LINUS.md)

## Goal

See processes the way the kernel does, and see system calls the way user space
makes them.

## You write

- `mycat.c`, `zombie.c` (user space)
- `tasks.c`, `Makefile` (module)

## Steps

1. `mycat.c`: copy a file to stdout using only `open()`, `read()`, `write()`,
   `close()`; no `stdio.h`. Run `strace ./mycat /etc/hostname` and match each
   line to your code.
2. `zombie.c`: `fork()`; the child exits immediately; the parent sleeps 30
   seconds without calling `wait()`. Find the zombie with `ps -o pid,ppid,stat,comm`.
3. `tasks.c` module: on load, walk every process with `for_each_process()`
   inside `rcu_read_lock()` / `rcu_read_unlock()` and print: PID, TGID, parent
   PID, state letter, number of threads, `comm`.
4. Load it while `zombie` is running. Find your zombie in the output.
5. Stretch: instead of printing on load, create `/proc/newlinus_tasks` with
   `proc_create_single()` and print from a `seq_file` show function, so
   `cat /proc/newlinus_tasks` gives a fresh list each time.

## Done when

- [ ] `mycat` works and its `strace` has no calls you can't explain.
- [ ] The module's output includes PID 1, `kthreadd` (PID 2) and your zombie
      with state `Z`.
- [ ] You can explain why the walk needs `rcu_read_lock()`.

## Hints

- `include/linux/sched/signal.h`: `for_each_process()`, `get_nr_threads()`
- `include/linux/pid.h`: `task_pid_nr()`, `task_tgid_nr()`
- `include/linux/sched.h`: `task_state_to_char()`, `p->comm`, `p->real_parent`
- `include/linux/proc_fs.h`, `include/linux/seq_file.h` for the stretch goal.

## C review: K&R chapter 4 and 8.1-8.3

Scope, `static`, header files, the preprocessor (4.11), and the UNIX system
interface. Read `mycat.c` against section 8.2.
