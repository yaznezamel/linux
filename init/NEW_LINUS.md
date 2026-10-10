# The New Linus: `init/`

Week 1 (Oct 12-18): boot. Lab: [`labs/01-hello`](../labs/01-hello/NEW_LINUS.md)

## What lives here

The architecture-independent part of boot. The bootloader and `arch/x86/boot/`
get the CPU into 64-bit mode and jump to C; from `start_kernel()` on, the code in
this folder sets up every subsystem and finally starts the first user-space
process, PID 1.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `init/main.c` | `start_kernel()` | Read the call list top to bottom. Each line brings up one subsystem: `setup_arch()`, `mm_core_init()`, `sched_init()`, `rcu_init()`, `init_IRQ()`, `softirq_init()`, `time_init()`, `console_init()`, ... The order is the dependency graph of the kernel. |
| `init/main.c` | `rest_init()` | Creates the first two kernel threads: `kernel_init` (becomes PID 1) and `kthreadd` (PID 2, parent of every kernel thread). |
| `init/main.c` | `kernel_init()`, `kernel_init_freeable()` | Runs the initcalls, calls `free_initmem()`, then execs `/init` from the initramfs or falls back to `/sbin/init`. |
| `init/main.c` | `do_initcalls()` | Runs every function registered with an `*_initcall()` macro, level by level. |
| `include/linux/init.h` | `__init`, `device_initcall()`, `late_initcall()` | How a function lands in an init section and at which boot stage it runs. A built-in driver's `module_init()` becomes a `device_initcall()` (`include/linux/module.h`). |
| `init/initramfs.c` | `populate_rootfs()` | Unpacks the initramfs (a cpio archive) into the in-memory root filesystem. |
| `init/do_mounts.c` | `prepare_namespace()` | Mounts the real root filesystem from `root=` when there is no initramfs `/init`. |
| `kernel/printk/printk.c` | `vprintk_emit()` | Where every `printk()`/`pr_info()` ends up: a ring buffer you read with `dmesg`. |

## Watch it in the VM

```sh
vng --append initcall_debug     # boot with every initcall logged
dmesg | head -5                 # "Linux version ...", "Command line: ..."
dmesg | grep -c 'calling '      # how many initcalls ran
dmesg | grep 'Freeing unused kernel image'   # __init code being thrown away
cat /proc/cmdline
ps -o pid,comm -p 1,2           # PID 1 here is virtme-ng-init, PID 2 is kthreadd
```

## Words to own

- **start_kernel()**: first generic C function of the kernel.
- **initcall**: a function registered to run at a fixed boot stage.
- **initramfs**: cpio archive unpacked into RAM; holds early user space and `/init`.
- **PID 1**: the first user-space process; it adopts orphaned processes.
- **kthreadd**: PID 2; every kernel thread is forked from it.
- **`__init` section**: code used only during boot and freed afterwards.
- **printk ring buffer**: fixed-size kernel log buffer behind `dmesg`.

## Answer in your notes

1. Why must `mm_core_init()` run before `sched_init()`?
2. Which process is PID 1 inside `vng`, and why is it not systemd?
3. How much memory does the "Freeing unused kernel image" line report on your build?
4. What is the difference between `module_init()` in a loadable module and in
   built-in code?
