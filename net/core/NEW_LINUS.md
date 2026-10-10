# The New Linus: `net/core/`

Weeks 8-9 (Nov 30-Dec 13): networking. Labs:
[`labs/08-mynic`](../../labs/08-mynic/NEW_LINUS.md),
[`labs/09-netfilter`](../../labs/09-netfilter/NEW_LINUS.md)

## What lives here

The protocol-independent middle of the stack: the socket buffer, the device
layer that sits between drivers and protocols, NAPI and the receive softirq,
neighbour (ARP/NDP) resolution, rtnetlink (what `ip` talks to), and network
namespaces. Function-by-function call chains for both directions are in
[`docs/networking/packet-path.md`](../../docs/networking/packet-path.md).

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `Documentation/networking/skbuff.rst` | whole doc | Head, data, tail, end; headroom and tailroom. |
| `skbuff.c` | `__alloc_skb()`, `skb_put()`, `skb_push()`, `skb_pull()`, `consume_skb()` | Allocate a buffer, then move pointers instead of copying data as headers are added or stripped. |
| `dev.c` | `register_netdev()`, `register_netdevice()` | How a driver's device becomes visible. |
| `dev.c` | `__dev_queue_xmit()` | Transmit entry point: qdisc, then the driver. |
| `dev.c` | `net_rx_action()`, `__napi_poll()` | The `NET_RX` softirq polling drivers in batches (NAPI). |
| `dev.c` | `netif_receive_skb()`, `__netif_receive_skb_core()` | Taps (tcpdump), then hand to the protocol (`ip_rcv()` for IPv4). |
| `neighbour.c` | `neigh_resolve_output()` | Queue the packet while ARP resolves the next hop's MAC address. |
| `rtnetlink.c` | `rtnl_newlink()` | What `ip link add` calls. |
| `net_namespace.c` | `copy_net_ns()` | A new network namespace: its own devices, routes, sockets. Containers. |

## Watch it in the VM

```sh
cat /proc/net/softnet_stat                  # per-CPU: processed, dropped, time_squeeze
ip netns add red && ip netns exec red ip link   # a namespace has only lo
tc qdisc show                               # queueing discipline per device
cat /proc/net/dev
```

## Words to own

- **sk_buff (skb)**: packet buffer plus metadata; headers are pushed and pulled,
  not copied.
- **headroom / tailroom**: free space before and after the data.
- **NAPI**: interrupt once, then poll with interrupts off until the queue is empty.
- **softnet / backlog**: per-CPU receive queues and statistics.
- **GRO / GSO**: merge small packets on receive / split large ones as late as
  possible on transmit.
- **qdisc**: queueing discipline; the scheduler for outgoing packets (`tc`).
- **netns**: network namespace, an isolated copy of the network stack.
- **rtnetlink**: netlink family for configuring links, addresses and routes.
- **time_squeeze**: NAPI ran out of budget with work left; a sign of overload.

## Answer in your notes

1. Why does `skb_push()` exist? Which layer calls it on transmit?
2. What does the third column of `/proc/net/softnet_stat` count?
3. What does a freshly created network namespace contain?
