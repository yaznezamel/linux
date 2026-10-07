# Roadmap

Six phases, about 20 weeks at 6-8 hours a week. The weeks are a guide, not a
deadline: move on when the deliverable for a phase is done. Status is tracked in
Notion; this file holds the technical detail.

Each phase serves the three goals: **C** (write code), **Linux** (understand a
subsystem), **Contribute/Share** (produce something you can show).

---

## Phase 0 - Set up the workbench (week 1)

Read [setup.md](setup.md) and get the loop of *edit → build → boot → test*
under five minutes.

- [ ] Build a kernel from this tree with a small config.
- [ ] Boot it in QEMU (virtme-ng) and see your own `CONFIG_LOCALVERSION`
      string in `uname -r`.
- [ ] Set up code navigation (`make compile_commands.json` + clangd, or
      `make cscope tags`).
- [ ] Read `Documentation/process/howto.rst`.

**Deliverable:** `uname -r` inside QEMU shows a kernel you built.

---

## Phase 1 - C through kernel code (weeks 2-4)

Read [c-in-the-kernel.md](c-in-the-kernel.md) alongside each exercise.

- [ ] Hello-world out-of-tree module (`module_init`, `module_exit`,
      `pr_info`, `MODULE_LICENSE`). Guide: `Documentation/kbuild/modules.rst`.
- [ ] Add a `module_param()` and read it back from
      `/sys/module/<name>/parameters/`.
- [ ] A module that keeps a `struct list_head` list of structs, adds, walks
      and frees them. Read `include/linux/list.h` and
      `include/linux/container_of.h` first.
- [ ] Expose a counter through debugfs (`debugfs_create_u32()`).
- [ ] Study `samples/kfifo/` and `samples/kobject/`, then rewrite one from
      memory.

**Deliverable:** three small modules in a personal repo, each with a short
README of what you learned.

---

## Phase 2 - Kernel core concepts (weeks 5-8)

The networking code depends on all of this, so it comes first.

| Topic | Read | Practice |
| --- | --- | --- |
| Memory allocation | `Documentation/core-api/memory-allocation.rst` | `kmalloc`/`kfree`, `kzalloc`, a `kmem_cache` |
| Locking | `Documentation/locking/` | Protect your Phase 1 list with a spinlock, then a mutex; explain when each is allowed |
| RCU | `Documentation/RCU/whatisRCU.rst` | Convert the list reader to RCU |
| Deferred work | `Documentation/core-api/workqueue.rst` | Workqueue + timer that updates a counter |
| Syscalls | `Documentation/process/adding-syscalls.rst` | Trace `sendto` with ftrace |
| Debugging | `Documentation/dev-tools/kasan.rst`, `Documentation/trace/ftrace.rst` | Introduce a use-after-free in your module and let KASAN catch it |

- [ ] Character device module (`misc_register()`), read/write from user space.
- [ ] Use `function_graph` tracing to see what `ping 127.0.0.1` calls.

**Deliverable:** a char device module and a written explanation of spinlock vs
mutex vs RCU, with your own example.

---

## Phase 3 - Networking deep dive (weeks 9-16)

The core of the plan. Follow the reading order in
[networking/README.md](networking/README.md) and the walkthrough in
[networking/packet-path.md](networking/packet-path.md).

- [ ] **Week 9-10: the data structures.** `struct sk_buff`
      (`include/linux/skbuff.h`, `Documentation/networking/skbuff.rst`) and
      `struct net_device` (`include/linux/netdevice.h`,
      `Documentation/networking/netdevices.rst`).
- [ ] **Week 10-11: the smallest driver.** Read `drivers/net/dummy.c`
      (200 lines) end to end, then write your own virtual netdev module that
      counts packets per protocol and reports them through `ethtool -S` or
      debugfs.
- [ ] **Week 12: two devices that talk.** Read `drivers/net/loopback.c`, then
      the transmit path in `drivers/net/veth.c`. Explain how a packet sent on
      one end of a veth pair arrives on the other.
- [ ] **Week 13-14: the life of a packet.** Trace `ping` and a UDP `sendto()`
      through the stack with ftrace; check every function against
      [networking/packet-path.md](networking/packet-path.md).
- [ ] **Week 15: hook into the stack.** Write a netfilter module
      (`nf_register_net_hook()`) that counts or drops ICMP echo requests.
- [ ] **Week 16: test like upstream does.** Build and run
      `tools/testing/selftests/net/` and read one test that covers code you
      studied. Look at `drivers/net/netdevsim/`, the fake device upstream uses
      to test driver APIs.

**Deliverables:** your virtual NIC module, your netfilter module, and a blog
post "The life of a packet in Linux" built from your own traces.

---

## Phase 4 - First upstream contribution (weeks 17-20)

Read [contributing.md](contributing.md) first.

- [ ] Configure `git send-email` and send a test patch to yourself.
- [ ] Pick a target (in order of ease): a fix in `Documentation/`, a
      `drivers/staging/` cleanup, a selftest improvement, a real bug fix.
- [ ] Run `scripts/checkpatch.pl --strict` and `scripts/get_maintainer.pl` on
      the patch.
- [ ] Send it, answer review, send v2 if asked.
- [ ] Subscribe to (or read on lore.kernel.org) `netdev` for a few weeks to
      learn how reviews work before sending networking patches.

**Deliverable:** one patch accepted into a maintainer tree.

---

## Phase 5 - Share it (ongoing)

- [ ] One short post per phase (LinkedIn, blog or dev.to).
- [ ] Keep [brag-sheet.md](brag-sheet.md) current with numbers you measured.
- [ ] Give a 10-minute talk to your team: "What happens when you ping".
- [ ] Add accepted patches to the Portfolio and Projects database in Notion,
      with the lore.kernel.org link.
