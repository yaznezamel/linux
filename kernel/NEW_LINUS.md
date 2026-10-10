# The New Linus: `kernel/`

Week 3 (Oct 26-Nov 1): processes and system calls. Lab:
[`labs/03-tasks`](../labs/03-tasks/NEW_LINUS.md). Subfolders have their own
guides: [`sched/`](sched/NEW_LINUS.md) (week 4), [`locking/`](locking/NEW_LINUS.md),
[`rcu/`](rcu/NEW_LINUS.md), [`irq/`](irq/NEW_LINUS.md) (week 6),
[`bpf/`](bpf/NEW_LINUS.md) (week 10).

## What lives here

The core that is not memory, files or networking: process creation and exit,
PIDs, signals, timers, the scheduler, locking, RCU, interrupts, printk, BPF.
For week 3 you only need process lifetime.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `include/linux/sched.h` | `struct task_struct` | The process descriptor. Find `pid`, `comm`, `__state`, `mm`, `parent`, `children`, `files`. Don't read all 700 lines of it; find those fields. |
| `kernel/sys.c` | `SYSCALL_DEFINE0(getpid)` | The smallest real system call. Note it returns the thread group id, not the thread id. |
| `kernel/fork.c` | `SYSCALL_DEFINE0(fork)` → `kernel_clone()` → `copy_process()` | `fork()`, `vfork()` and `clone()` all funnel into `kernel_clone()`. |
| `kernel/fork.c` | `dup_task_struct()`, `copy_mm()` | Copying the descriptor and sharing or copying the address space. Copy-on-write starts here. |
| `kernel/pid.c` | `alloc_pid()` | How a PID number is picked, per PID namespace (containers). |
| `kernel/exit.c` | `do_exit()`, `exit_notify()` | Tear-down, and how a process becomes a zombie (`EXIT_ZOMBIE`). |
| `kernel/exit.c` | `SYSCALL_DEFINE4(wait4)`, `do_wait()` | How the parent reaps the zombie. |

## Watch it in the VM

```sh
strace -f -e trace=clone,clone3,execve,wait4 sh -c 'ls >/dev/null'
cat /proc/self/status | head -10          # Name, State, Tgid, Pid, PPid
ls /proc/self/task                         # one entry per thread

cd /sys/kernel/tracing
echo kernel_clone > set_ftrace_filter
echo function > current_tracer
sh -c true
echo 0 > tracing_on; grep -v '^#' trace | head
```

`strace` is on your Mint host; `vng` shares the host filesystem, so it works
inside the VM too.

## Words to own

- **task_struct**: the kernel's descriptor for one thread; a process is a group
  of them sharing an `mm`.
- **TGID vs PID**: what user space calls a PID is the kernel's thread group id.
- **copy-on-write (COW)**: after `fork()` parent and child share pages
  read-only; the first write to a page copies it.
- **zombie**: a task that has exited but whose parent has not called `wait()`.
- **kernel thread**: a task with no user address space (`mm == NULL`).
- **PID namespace**: a separate PID numbering, which is how a container's
  first process is PID 1.

## Answer in your notes

1. What does `getpid()` return in a thread that is not the main thread, and why?
2. Which `clone` flags make `kernel_clone()` create a thread instead of a process?
3. Write a C program that leaves a zombie for 30 seconds; find it with
   `ps -o pid,stat,comm` and explain the `Z`.
