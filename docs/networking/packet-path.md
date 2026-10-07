# The life of a packet (IPv4)

Function names below are from this tree (v7.3-rc6). Use them as `ftrace`
filters and `git grep` targets. Netfilter hook points are marked **[NF]**.

## Transmit: `sendto()` on a UDP socket

```text
user space: sendto(fd, buf, len, 0, &addr, sizeof(addr))
  │
  ├─ __sys_sendto()                      net/socket.c
  │   └─ sock_sendmsg() → sock_sendmsg_nosec()
  │       └─ inet_sendmsg()              net/ipv4/af_inet.c   (proto_ops of the socket)
  │           └─ udp_sendmsg()           net/ipv4/udp.c       (route lookup, build skb)
  │               └─ udp_send_skb()      add UDP header + checksum
  │                   └─ ip_send_skb()   net/ipv4/ip_output.c
  │                       └─ ip_local_out() → __ip_local_out()      [NF] LOCAL_OUT
  │                           └─ dst_output() → ip_output()         [NF] POST_ROUTING
  │                               └─ ip_finish_output() → ip_finish_output2()
  │                                   └─ neigh_output()             include/net/neighbour.h (ARP/next hop)
  │                                       └─ dev_queue_xmit()       include/linux/netdevice.h
  │                                           └─ __dev_queue_xmit() net/core/dev.c
  │                                               └─ __dev_xmit_skb() → sch_direct_xmit()   (qdisc, net/sched/)
  │                                                   └─ dev_hard_start_xmit() → xmit_one()
  │                                                       └─ netdev_start_xmit()
  │                                                           └─ ops->ndo_start_xmit()   the driver
  ▼
driver: dummy_xmit() frees it, loopback_xmit() turns it into a receive,
        a real NIC driver maps it for DMA and rings the hardware doorbell
```

## Receive: a packet arrives on a NIC

```text
hardware raises an interrupt
  │
  ├─ driver IRQ handler → napi_schedule()        (do as little as possible in hardirq)
  │
  ├─ NET_RX_SOFTIRQ → net_rx_action()             net/core/dev.c
  │   └─ napi_poll() → __napi_poll() → driver's poll function
  │       └─ napi_gro_receive()                   (GRO merges segments, include/net/gro.h)
  │           └─ gro_normal_list() → netif_receive_skb_list_internal()
  │               → ... → __netif_receive_skb_core()
  │               ├─ taps: tcpdump/AF_PACKET see the packet here
  │               └─ deliver_skb() → packet_type handler for ETH_P_IP
  │                   └─ ip_rcv()                 net/ipv4/ip_input.c   (registered in af_inet.c)
  │                       └─ ip_rcv_core()        header checks         [NF] PRE_ROUTING
  │                           └─ ip_rcv_finish() → ip_rcv_finish_core()
  │                               └─ ip_route_input_noref()   local, forward, or drop?
  │                                   └─ dst_input() → ip_local_deliver()   [NF] LOCAL_IN
  │                                       └─ ip_local_deliver_finish()
  │                                           └─ ip_protocol_deliver_rcu()
  │                                               ├─ icmp_rcv()   net/ipv4/icmp.c
  │                                               │   └─ icmp_echo() → icmp_reply()   (ping answered here)
  │                                               └─ udp_rcv()    net/ipv4/udp.c → socket receive queue
  ▼
user space: recvfrom() wakes up and copies the data out
```

Loopback is a shortcut: `loopback_xmit()` calls `__netif_rx()`, which queues
the skb on the per-CPU backlog (`enqueue_to_backlog()`); `process_backlog()`
then runs as a NAPI poll function and joins the receive path at
`__netif_receive_skb()`.

## How the layers find each other

| Lookup | Registered in | Used by |
| --- | --- | --- |
| EtherType → `ip_rcv()` | `dev_add_pack(&ip_packet_type)` in `net/ipv4/af_inet.c` | `__netif_receive_skb_core()` |
| IP protocol number → `icmp_rcv()` / `udp_rcv()` | `struct net_protocol` entries in `net/ipv4/af_inet.c` | `ip_protocol_deliver_rcu()` |
| Socket call → protocol | `struct proto_ops inet_dgram_ops` and `struct proto udp_prot` | `sock_sendmsg()` |
| Device → driver | `dev->netdev_ops` | `netdev_start_xmit()` |

## Trace it yourself

Inside the VM (needs `CONFIG_FUNCTION_GRAPH_TRACER`):

```sh
cd /sys/kernel/tracing
echo icmp_rcv > set_graph_function
echo function_graph > current_tracer
echo 1 > tracing_on
ping -c1 127.0.0.1
echo 0 > tracing_on
less trace
```

Swap `icmp_rcv` for `__sys_sendto` and run a small UDP sender to see the
transmit side. Write down every function in the trace that is **not** in the
diagrams above and find out what it does.
