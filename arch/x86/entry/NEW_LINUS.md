# The New Linus: `arch/x86/entry/`

Week 3 (Oct 26-Nov 1): processes and system calls. Lab:
[`labs/03-tasks`](../../../labs/03-tasks/NEW_LINUS.md)

## What lives here

The border between user space and the kernel on x86-64. Every system call,
interrupt and exception enters the kernel through code in this folder, saves the
user registers, and leaves through it again.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `syscalls/syscall_64.tbl` | lines for `read` (0), `write` (1), `getpid` (39), `clone` (56), `fork` (57) | The system call numbers. This table generates the dispatch code at build time. |
| `entry_64.S` | `entry_SYSCALL_64` | Where the CPU lands after the `syscall` instruction: swap to the kernel stack, save registers into `struct pt_regs`, call C. Read the comment block above it; skip the details of the assembly. |
| `syscall_64.c` | `do_syscall_64()`, `x64_sys_call()` | Range-check the number and dispatch. The comment says `sys_call_table[]` is no longer used for dispatch: since 2024 a generated `switch` replaces the indirect call through the table, as a defence against Branch History Injection (Spectre-BHI). |
| `vdso/` and `lib/vdso/gettimeofday.c` | `do_hres()` | The vDSO: code the kernel maps into every process so `clock_gettime()` runs without entering the kernel. |

## Watch it in the VM

```sh
grep -E '^(0|1|39|57)\s' arch/x86/entry/syscalls/syscall_64.tbl
strace -c ls > /dev/null                  # syscall counts for one command
grep vdso /proc/self/maps                 # the vDSO mapping
```

## Words to own

- **system call**: a controlled entry into the kernel; on x86-64 via the
  `syscall` instruction, number in `rax`, arguments in `rdi, rsi, rdx, r10, r8, r9`.
- **pt_regs**: the saved user register state on the kernel stack.
- **kernel stack**: a small per-thread stack (16 KiB on x86-64, 32 KiB with
  KASAN) used while in the kernel.
- **vDSO**: virtual dynamic shared object; kernel code run in user mode for
  fast time queries.
- **Spectre-BHI**: a speculative-execution attack that abuses indirect branches;
  the reason for the `switch`-based dispatch.

## Answer in your notes

1. Why is the fourth argument in `r10` and not `rcx` like the normal C calling
   convention?
2. Run `strace -c` on `date`. Does `clock_gettime` show up? Why or why not?
3. Follow `getpid` from `syscall_64.tbl` to `kernel/sys.c`: list every function
   on the way.
