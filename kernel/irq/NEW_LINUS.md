# The New Linus: `kernel/irq/` and `kernel/softirq.c`

Week 6 (Nov 16-22): concurrency and interrupts. Lab:
[`labs/06-locking`](../../labs/06-locking/NEW_LINUS.md)

## What lives here

Generic interrupt handling. A device raises an interrupt; the CPU stops what it
is doing and runs a handler with interrupts disabled. Handlers must be short, so
work is split in two:

- **top half (hardirq)**: acknowledge the device, grab the data, schedule the rest.
- **bottom half**: softirqs (`kernel/softirq.c`), tasklets, threaded IRQs, or
  workqueues (`kernel/workqueue.c`) do the slower work later.

Networking lives on this split: the NIC interrupt only schedules NAPI, and
packets are processed in the `NET_RX` softirq (week 8).

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `include/linux/interrupt.h` | `request_irq()`, enum starting `HI_SOFTIRQ` | How a driver registers a handler; the fixed list of softirq types (`NET_TX`, `NET_RX`, `TIMER`, `RCU`, ...). |
| `kernel/irq/manage.c` | `request_threaded_irq()` | Registering a handler, optionally with a threaded bottom half. |
| `kernel/irq/handle.c` | `handle_irq_event()` | Calls every handler registered on that interrupt line. |
| `kernel/irq/chip.c` | `handle_edge_irq()`, `handle_fasteoi_irq()` | Flow handlers: edge vs level-triggered interrupts. Skim. |
| `kernel/softirq.c` | `irq_exit()`, `invoke_softirq()`, `handle_softirqs()` | Softirqs run on the way out of a hardirq; if there is too much work, `ksoftirqd` takes over. |
| `kernel/softirq.c` | `open_softirq()`, `raise_softirq()` | Registering and triggering a softirq. `net/core/dev.c` calls `open_softirq(NET_RX_SOFTIRQ, net_rx_action)`. |
| `kernel/workqueue.c` | `queue_work_on()` | Deferred work in process context, where sleeping is allowed. |

## Watch it in the VM

```sh
cat /proc/interrupts                       # per-CPU count per IRQ line
cat /proc/softirqs                         # per-CPU count per softirq type
ps -e -o pid,comm | grep -E 'ksoftirqd|kworker' | head
```

Run `cat /proc/softirqs` twice around a `ping -c 100 -i 0.01 127.0.0.1` and
compare the `NET_RX` row.

## Words to own

- **IRQ**: interrupt request; a hardware signal to the CPU.
- **hardirq / top half**: the handler run with interrupts off.
- **softirq / bottom half**: deferred, interrupts on, still atomic (no sleeping).
- **ksoftirqd**: per-CPU thread that runs softirqs under load.
- **threaded IRQ**: handler run in a kernel thread, so it may sleep.
- **workqueue / kworker**: deferred work in process context.
- **interrupt coalescing**: hardware batching several events into one interrupt.

## Answer in your notes

1. Why can a softirq not sleep, even though interrupts are enabled while it runs?
2. On your VM, which CPU handles most `NET_RX` softirqs during the ping test?
3. When would you pick a workqueue over a softirq?
