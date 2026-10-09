# Networking: map and reading order

Networking is one of the largest parts of the kernel: about 1.3 million lines of
C in `net/` and 6.6 million counting `drivers/net/` (measured on this tree,
v7.3-rc6). Don't try to read it top-down. Start from the two central data
structures, then follow one packet.

## The map

| Layer | Directory | What lives there |
| --- | --- | --- |
| System calls | `net/socket.c` | `socket()`, `sendto()`, `recvfrom()` entry points |
| Socket families | `net/ipv4/af_inet.c`, `net/unix/`, `net/packet/` | Glue between the socket API and protocols |
| Transport | `net/ipv4/tcp*.c`, `net/ipv4/udp.c`, `net/ipv4/ping.c`, `net/ipv4/raw.c` | TCP, UDP, ICMP sockets, raw sockets |
| Network | `net/ipv4/ip_input.c`, `ip_output.c`, `route.c`, `icmp.c`, `net/ipv6/` | IP send/receive, routing, ICMP |
| Neighbour | `net/core/neighbour.c`, `net/ipv4/arp.c` | ARP, next-hop resolution |
| Device core | `net/core/dev.c` | Queuing, NAPI, softirq, handing packets to protocols |
| Buffers | `net/core/skbuff.c`, `include/linux/skbuff.h` | `struct sk_buff` |
| Packet filtering | `net/netfilter/` | iptables/nftables hooks, conntrack, NAT |
| Traffic control | `net/sched/` | qdiscs, `tc` |
| Programmable path | `net/core/filter.c`, `net/xdp/`, `kernel/bpf/` | eBPF, XDP |
| Configuration | `net/core/rtnetlink.c`, `net/netlink/` | What `ip link` / `ip addr` talk to |
| Drivers | `drivers/net/` | Virtual devices (`dummy`, `veth`, `tun`) and real NICs (`ethernet/`) |
| Tests | `tools/testing/selftests/net/` | Upstream networking tests |

## Reading order

Each step: read the code, run it, write a note with
[the template](../templates/study-notes.md).

| # | Read | Size | Why |
| --- | --- | --- | --- |
| 1 | `Documentation/networking/skbuff.rst`, then `include/linux/skbuff.h` (the struct and `skb_put`/`skb_push`/`skb_pull`) | Doc + header | Every function in the stack takes an `sk_buff` |
| 2 | `Documentation/networking/netdevices.rst`, `struct net_device_ops` in `include/linux/netdevice.h` | Doc + header | The interface every driver implements |
| 3 | `drivers/net/dummy.c`, guided: [01-dummy-driver.md](01-dummy-driver.md) | 200 lines | A complete driver: setup, xmit, stats, rtnl_link_ops, module params |
| 4 | `drivers/net/loopback.c` | 292 lines | A device that transmits back into receive (`loopback_xmit()` → `__netif_rx()`) |
| 5 | `drivers/net/veth.c` (`veth_xmit()` and NAPI parts) | ~2000 lines | Two linked devices; how containers get networking |
| 6 | `Documentation/networking/napi.rst` | Doc | How receive is batched and moved to softirq |
| 7 | `net/core/dev.c`: `__dev_queue_xmit()`, `__netif_receive_skb_core()`, `net_rx_action()` | Selected functions | The center of the stack |
| 8 | `net/ipv4/ip_input.c`, `net/ipv4/icmp.c` (`icmp_rcv()`, `icmp_echo()`) | ~700 + selected | How a ping is answered |
| 9 | `net/ipv4/udp.c`: `udp_sendmsg()`, `udp_rcv()` | Selected functions | The simplest transport protocol |
| 10 | `net/netfilter/core.c`: `nf_register_net_hook()`, `nf_hook_slow()` | Selected functions | Where iptables/nftables plug in |
| 11 | `drivers/net/netdevsim/` | Directory | Fake device used to test driver APIs, a common place for upstream tests |
| 12 | `net/ipv4/tcp_input.c` | ~7800 lines | Only after everything above |

Then follow the full path in [packet-path.md](packet-path.md).

## Hands-on exercises

1. **Own virtual NIC.** Copy the structure of `dummy.c` into an out-of-tree
   module `mynic.c`. Instead of dropping packets, count them per
   `skb->protocol` and print the counts from debugfs. Create it with
   `ip link add mynic0 type mynic` (needs `rtnl_link_ops`), bring it up, send
   traffic with `ping -I mynic0`.
2. **Echo device.** Make `mynic` behave like loopback: hand transmitted
   packets back to the stack with `__netif_rx()`. What breaks, and why does
   `loopback.c` call `skb_orphan()` first?
3. **ICMP counter with netfilter.** Register an `NF_INET_LOCAL_IN` hook that
   counts ICMP echo requests and drops every second one. Verify with `ping`.
4. **Trace a ping.** With `function_graph` and `set_graph_function` set to
   `icmp_rcv`, run `ping -c1 127.0.0.1` and match the trace to
   [packet-path.md](packet-path.md).
5. **Run the selftests.**
   `make -C tools/testing/selftests TARGETS=net run_tests` inside your VM; read
   one failing or skipped test and find out why.

## Key documents

- `Documentation/networking/index.rst` - the full list
- `Documentation/networking/kapi.rst` - API reference
- `Documentation/networking/scaling.rst` - RSS, RPS, RFS, XPS
- `Documentation/networking/filter.rst` - classic BPF and eBPF for sockets
- `Documentation/process/maintainer-netdev.rst` - **how the netdev
  maintainers work; read before sending a networking patch**

## Community

- Mailing list: `netdev@vger.kernel.org` (archive:
  <https://lore.kernel.org/netdev/>)
- Patch tracking: <https://patchwork.kernel.org/project/netdevbpf/list/>
- Trees: `net` (fixes) and `net-next` (new work), see `MAINTAINERS` entry
  `NETWORKING [GENERAL]`.
