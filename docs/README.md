# Linux Learning Journal

Personal notes for learning C and the Linux kernel by reading and changing this
fork of the kernel tree. This folder is **not** part of the upstream kernel; the
official kernel documentation lives in [`Documentation/`](../Documentation).

## Goals

1. **Learn C through practice** - read real kernel C and write small modules.
2. **Understand Linux in depth** - processes, memory, concurrency and, first of
   all, the networking stack.
3. **Contribute upstream and talk about it** - land patches in mainline and be
   able to explain, with real numbers, why Linux is a remarkable project.

## Where things live

| What | Where |
| --- | --- |
| Plan, milestone tracker, weekly reviews | Notion: [Second Brain → Projects → Linux Kernel Deep Dive](https://app.notion.com/p/3f285e551cdd817db5f6d8db256f5dd6) |
| Technical notes that belong next to the code | This `docs/` folder |
| Official kernel documentation | [`Documentation/`](../Documentation) |

## Contents

| File | What it is |
| --- | --- |
| [ROADMAP.md](ROADMAP.md) | Phases, exercises and deliverables |
| [setup.md](setup.md) | Build, boot and navigate the kernel |
| [c-in-the-kernel.md](c-in-the-kernel.md) | C idioms the kernel uses everywhere |
| [networking/README.md](networking/README.md) | Map of the networking code and a reading order |
| [networking/packet-path.md](networking/packet-path.md) | The life of a packet, function by function |
| [contributing.md](contributing.md) | How to send a first patch, and where |
| [brag-sheet.md](brag-sheet.md) | Facts and numbers for talking about Linux |
| [learning-log.md](learning-log.md) | Dated log of sessions |
| [templates/study-notes.md](templates/study-notes.md) | Template for studying one file or subsystem |

## Working rules

- Keep notes short and tied to a file path and function name, for example
  `net/ipv4/icmp.c:icmp_echo()`, so they stay useful as the code moves.
- Every study session ends with one line in [learning-log.md](learning-log.md)
  and a status update in Notion.
- Keep this folder off any branch that you send upstream. Branch patches from
  upstream `master`, never from a branch that contains `docs/`
  (see [contributing.md](contributing.md)).
