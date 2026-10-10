# Lab 10: bpf

Week 10 (Dec 14-20). Read first: [`kernel/bpf/NEW_LINUS.md`](../../kernel/bpf/NEW_LINUS.md)

## Goal

Observe the kernel without writing a module: the same questions as earlier
labs, answered with bpftrace programs that the verifier checks and the JIT
compiles.

## You write

- `syscalls.bt`, `udp_sizes.bt`, `xmit.bt` (bpftrace scripts)
- `RESULTS.md`

## Steps

Install on the host: `sudo apt install bpftrace`. Boot with `vng --user root`.

1. `syscalls.bt`: count system calls per process name for 10 seconds, print
   the top 10. (`tracepoint:raw_syscalls:sys_enter`, `@[comm] = count()`,
   `interval:s:10 { exit(); }`.)
2. `udp_sizes.bt`: a histogram of the `len` argument of `udp_sendmsg()` while
   your lab 09 `udp_echo` client sends messages of different sizes
   (`kprobe:udp_sendmsg`, third argument).
3. `xmit.bt`: attach to your lab 08 driver's transmit function (or
   `dummy_xmit`) and print a histogram of `skb->len`. With BTF you can
   dereference `((struct sk_buff *)arg0)->len` directly.
4. Build bpftool from the tree on the host (`sudo apt install zlib1g-dev`,
   then `make -C tools/bpf/bpftool`; Mint's packaged bpftool is tied to Mint's
   kernel version). While a script runs, `tools/bpf/bpftool/bpftool prog list`,
   find your program, and dump its machine code with
   `tools/bpf/bpftool/bpftool prog dump jited id <ID>`.
5. Write in `RESULTS.md`: one thing each script showed that you did not expect.

## Done when

- [ ] All three scripts run and their output is in `RESULTS.md`.
- [ ] You can explain why loading these programs could not crash the kernel,
      while a buggy module from lab 06 could.

## C review: kernel coding style

Read `Documentation/process/coding-style.rst` end to end, then run
`scripts/checkpatch.pl -f --no-tree` on every `.c` file in `labs/` and fix
everything it reports. This is the last step before your first real patch.
