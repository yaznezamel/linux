# The New Linus

A 12-week plan, Oct 12 to Dec 31 2026, to learn Linux internals, review C, send
a first kernel patch, and be able to talk about all of it with precision.

This folder and every `NEW_LINUS.md` file are personal notes in this fork. They
are not part of the upstream kernel and must never be sent upstream; the
official documentation is in [`Documentation/`](../Documentation).

## Start here

1. Open the plan in Notion: [The New Linus](https://app.notion.com/p/3f585e551cdd8190a421f75d3d8dd5e8).
   The **Weeks** table says what to do this week; open the row for the checklist.
2. The row lists folder guides. Each is a `NEW_LINUS.md` inside the kernel
   folder it describes. Read it with the code next to it.
3. The row names a lab. Your code goes in `labs/NN-name/`; the assignment is
   that folder's `NEW_LINUS.md`.
4. Notes go in `docs/notes/wNN.md` (create one per week).
5. On Sunday, update the week row and the **Lexicon** in Notion.

## Week by week

| Week | Dates | Folder guides | Lab |
| --- | --- | --- | --- |
| 1 Boot | Oct 12-18 | [`setup.md`](setup.md), [`init/`](../init/NEW_LINUS.md) | [`01-hello`](../labs/01-hello/NEW_LINUS.md) |
| 2 C and data structures | Oct 19-25 | [`include/linux/`](../include/linux/NEW_LINUS.md), [`lib/`](../lib/NEW_LINUS.md), [`c-in-the-kernel.md`](c-in-the-kernel.md) | [`02-list`](../labs/02-list/NEW_LINUS.md) |
| 3 Processes and syscalls | Oct 26-Nov 1 | [`kernel/`](../kernel/NEW_LINUS.md), [`arch/x86/entry/`](../arch/x86/entry/NEW_LINUS.md) | [`03-tasks`](../labs/03-tasks/NEW_LINUS.md) |
| 4 Scheduler | Nov 2-8 | [`kernel/sched/`](../kernel/sched/NEW_LINUS.md) | [`04-sched`](../labs/04-sched/NEW_LINUS.md) |
| 5 Memory | Nov 9-15 | [`mm/`](../mm/NEW_LINUS.md) | [`05-mm`](../labs/05-mm/NEW_LINUS.md) |
| 6 Concurrency and interrupts | Nov 16-22 | [`kernel/locking/`](../kernel/locking/NEW_LINUS.md), [`kernel/rcu/`](../kernel/rcu/NEW_LINUS.md), [`kernel/irq/`](../kernel/irq/NEW_LINUS.md) | [`06-locking`](../labs/06-locking/NEW_LINUS.md) |
| 7 Files and devices | Nov 23-29 | [`fs/`](../fs/NEW_LINUS.md), [`drivers/char/`](../drivers/char/NEW_LINUS.md) | [`07-chardev`](../labs/07-chardev/NEW_LINUS.md) |
| 8 Networking I | Nov 30-Dec 6 | [`drivers/net/`](../drivers/net/NEW_LINUS.md), [`net/core/`](../net/core/NEW_LINUS.md), [`networking/01-dummy-driver.md`](networking/01-dummy-driver.md) | [`08-mynic`](../labs/08-mynic/NEW_LINUS.md) |
| 9 Networking II | Dec 7-13 | [`net/ipv4/`](../net/ipv4/NEW_LINUS.md), [`net/netfilter/`](../net/netfilter/NEW_LINUS.md), [`networking/packet-path.md`](networking/packet-path.md) | [`09-netfilter`](../labs/09-netfilter/NEW_LINUS.md) |
| 10 eBPF, tests, first patch | Dec 14-20 | [`kernel/bpf/`](../kernel/bpf/NEW_LINUS.md), [`tools/testing/selftests/net/`](../tools/testing/selftests/net/NEW_LINUS.md), [`drivers/staging/`](../drivers/staging/NEW_LINUS.md), [`contributing.md`](contributing.md) | [`10-bpf`](../labs/10-bpf/NEW_LINUS.md) |
| 11 Review and write-up | Dec 21-27 | [`brag-sheet.md`](brag-sheet.md) | patch v2, blog post |
| 12 Year-end | Dec 28-31 | [`brag-sheet.md`](brag-sheet.md) | talk, retro |

List every guide in the tree: `git ls-files '*NEW_LINUS.md'`

## Reference in this folder

| File | What it is |
| --- | --- |
| [setup.md](setup.md) | Build, boot and navigate the kernel; timings |
| [c-in-the-kernel.md](c-in-the-kernel.md) | C idioms the kernel uses everywhere |
| [networking/README.md](networking/README.md) | Map of the networking code |
| [networking/packet-path.md](networking/packet-path.md) | Transmit and receive call chains |
| [networking/01-dummy-driver.md](networking/01-dummy-driver.md) | Guided read of `dummy.c` with a traced experiment |
| [contributing.md](contributing.md) | Patch workflow and netdev rules |
| [brag-sheet.md](brag-sheet.md) | Numbers measured on this tree, talking points |
| [learning-log.md](learning-log.md) | Dated one-line session log |
| [templates/study-notes.md](templates/study-notes.md) | Template for studying a file in depth |

## Rules

- Notes name a file and a function, for example `net/ipv4/icmp.c:icmp_echo()`.
- Patches for upstream are branched from upstream (`master`, `staging-next`,
  `net-next`), never from this branch. See [contributing.md](contributing.md).
