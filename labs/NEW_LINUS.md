# The New Linus: `labs/`

Plan and tracker: [The New Linus](https://app.notion.com/p/3f585e551cdd8190a421f75d3d8dd5e8) in Notion. Index of all guides: [`docs/README.md`](../docs/README.md).

Your code goes here, one folder per lab. Each folder's `NEW_LINUS.md` is the
assignment. You write the `.c` files and the `Makefile`; nothing is pre-written.

This folder is not part of the kernel build and must never go upstream.

| Lab | Week | Builds |
| --- | --- | --- |
| [`01-hello`](01-hello/NEW_LINUS.md) | 1 (Oct 12-18) | first module, module parameters |
| [`02-list`](02-list/NEW_LINUS.md) | 2 (Oct 19-25) | linked list of structs, `container_of` in user space |
| [`03-tasks`](03-tasks/NEW_LINUS.md) | 3 (Oct 26-Nov 1) | walk every process; syscall-only `cat` |
| [`04-sched`](04-sched/NEW_LINUS.md) | 4 (Nov 2-8) | measure CPU share vs nice value |
| [`05-mm`](05-mm/NEW_LINUS.md) | 5 (Nov 9-15) | page faults, slab cache, buddy pages |
| [`06-locking`](06-locking/NEW_LINUS.md) | 6 (Nov 16-22) | a real race, three fixes, a lockdep report |
| [`07-chardev`](07-chardev/NEW_LINUS.md) | 7 (Nov 23-29) | `/dev/newlinus` |
| [`08-mynic`](08-mynic/NEW_LINUS.md) | 8 (Nov 30-Dec 6) | your own virtual network device |
| [`09-netfilter`](09-netfilter/NEW_LINUS.md) | 9 (Dec 7-13) | ICMP counter and dropper |
| [`10-bpf`](10-bpf/NEW_LINUS.md) | 10 (Dec 14-20) | bpftrace instead of modules |

## One-time setup (week 1)

Enable everything the labs need in one go, so you don't rebuild every week:

```sh
cd ~/linux
scripts/config -e DUMMY -e VETH -e NET_NS \
    -e NETFILTER -e NETFILTER_ADVANCED -e NF_CONNTRACK \
    -e DEBUG_INFO_BTF -e BPF_EVENTS \
    -e PROVE_LOCKING -e DEBUG_ATOMIC_SLEEP \
    -e IKCONFIG -e IKCONFIG_PROC
make olddefconfig
vng --build
```

`PROVE_LOCKING` (lockdep) makes the kernel slower; that is fine in a VM and it
is what catches your locking bugs in week 6.

## Build and load a module

Every module lab uses the same loop. The `Makefile` in the lab folder is one line:

```make
obj-m += hello.o
```

```sh
cd ~/linux/labs/01-hello
make -C ~/linux M=$PWD modules          # build hello.ko against your kernel
cd ~/linux && vng --user root           # boot as root (insmod needs it)

# inside the VM (the current directory is ~/linux):
insmod labs/01-hello/hello.ko count=3
dmesg | tail
rmmod hello
```

Clean up with `make -C ~/linux M=$PWD clean`. The build outputs (`*.o`, `*.ko`,
`.*.cmd`) are already ignored by the kernel's `.gitignore`.

## Rules

- Write the code yourself. Read kernel code for patterns; don't copy whole files.
- Every lab must load and unload cleanly: no warnings in `dmesg`, no leaks.
- Run `scripts/checkpatch.pl -f --no-tree labs/NN-name/*.c` before calling a
  lab done. From week 10 on, zero warnings.
- Commit each finished lab with a message that says what you learned.
