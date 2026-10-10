# The New Linus: `kernel/bpf/`

Week 10 (Dec 14-20): eBPF, tests, first patch. Lab:
[`labs/10-bpf`](../../labs/10-bpf/NEW_LINUS.md)

## What lives here

eBPF: a small virtual machine inside the kernel. User space loads a program,
the **verifier** proves it terminates and only touches memory it is allowed to,
the **JIT** compiles it to native x86-64, and it is attached to a hook: a
kprobe, a tracepoint, a socket, XDP in a driver, a cgroup, or the scheduler
(sched_ext). It is how modern observability (bpftrace, Cilium, Falco) works
without loading kernel modules.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `syscall.c` | `SYSCALL_DEFINE5(bpf, ...)`, `bpf_prog_load()`, `map_create()` | One system call with a command number does everything: load programs, create maps, attach. |
| `verifier.c` | `bpf_check()`, `do_check()` | The verifier walks every path of the program and tracks the type and range of every register. 21,000+ lines; read the top-of-file comment and `do_check()`'s loop only. |
| `core.c` | `___bpf_prog_run()` | The interpreter, used when the JIT is off. Each opcode is a jump-table label. |
| `arch/x86/net/bpf_jit_comp.c` | `bpf_int_jit_compile()` | BPF bytecode to x86-64 machine code. |
| `helpers.c` | `BPF_CALL_2(bpf_map_lookup_elem, ...)` | Helpers are the kernel functions a program is allowed to call. |
| `hashtab.c`, `arraymap.c` | top of file | Maps: key/value stores shared between BPF programs and user space. |

## Watch it in the VM

Needs `CONFIG_BPF_SYSCALL`, `CONFIG_BPF_JIT`, `CONFIG_DEBUG_INFO_BTF` and
`CONFIG_BPF_EVENTS` (week 1 learning config) and `bpftrace` on the host
(`sudo apt install bpftrace`).

```sh
cat /proc/sys/net/core/bpf_jit_enable       # 1 = JIT on
bpftrace -e 'tracepoint:syscalls:sys_enter_openat { @[comm] = count(); }'
bpftrace -l 'kprobe:udp_*' | head           # attachable kernel functions
```

## Words to own

- **eBPF**: in-kernel virtual machine for safe, loadable programs.
- **verifier**: static analyser that rejects unsafe programs before they load.
- **JIT**: just-in-time compiler from BPF bytecode to native code.
- **map**: key/value storage shared with user space.
- **helper / kfunc**: kernel functions callable from BPF.
- **BTF**: BPF Type Format; compact type info so programs work across kernel builds.
- **CO-RE**: compile once, run everywhere, enabled by BTF.
- **kprobe / tracepoint / fentry**: dynamic, static, and BTF-based attach points.
- **XDP**: BPF run in the driver before an skb is allocated.

## Answer in your notes

1. Why does the verifier reject an unbounded loop?
2. What is the difference between a kprobe and a tracepoint, and which one can
   break when the kernel changes?
3. Count `openat` calls per process for 10 seconds with `bpftrace`. Which
   process wins on your VM?
