# The New Linus: `net/ipv4/`

Week 9 (Dec 7-13): networking II, the stack and netfilter. Lab:
[`labs/09-netfilter`](../../labs/09-netfilter/NEW_LINUS.md). Same week:
[`net/netfilter/`](../netfilter/NEW_LINUS.md).

## What lives here

IPv4 and everything that rides on it: IP input and output, routing (the FIB),
ARP, ICMP, UDP, TCP, raw sockets, and the `AF_INET` socket family that glues
them to the socket API. Follow one packet in each direction with
[`docs/networking/packet-path.md`](../../docs/networking/packet-path.md) open.

## Read, in this order

| File | Symbol | What to get out of it |
| --- | --- | --- |
| `af_inet.c` | `inet_create()`, `ip_packet_type`, the `net_protocol` entries | How `socket(AF_INET, SOCK_DGRAM, 0)` gets UDP ops, and how received packets find `ip_rcv()`, `icmp_rcv()`, `udp_rcv()`. |
| `ip_input.c` | `ip_rcv()` → `ip_rcv_finish()` → `ip_local_deliver()` | Header checks, routing decision, local delivery. Note the two `NF_HOOK` calls. |
| `icmp.c` | `icmp_rcv()`, `icmp_echo()` | Where a ping is answered. |
| `udp.c` | `udp_sendmsg()`, `udp_rcv()` | The simplest transport protocol, both directions. |
| `ip_output.c` | `ip_queue_xmit()`, `ip_output()` | Build the IP header, run the output hooks, hand to the neighbour layer. |
| `route.c` | `ip_route_output_key_hash()` | Output route lookup with its cache. |
| `fib_trie.c` | `fib_table_lookup()` | Longest-prefix match over an LC-trie. This is what `ip route` populates. |
| `arp.c` | `arp_rcv()`, `arp_send()` | MAC address resolution for IPv4. |
| `tcp_input.c` | file size only (`wc -l`) | 7,800 lines. Leave TCP for 2027. |

## Watch it in the VM

```sh
ip route; ip neigh                          # FIB and ARP cache
ss -uanp                                    # UDP sockets
cat /proc/net/snmp | grep -E '^(Ip|Icmp|Udp):'   # protocol counters
ping -c 2 127.0.0.1; grep '^Icmp:' /proc/net/snmp
```

Tracing a ping and a UDP send is lab 08's second half and week 9's homework:
`set_graph_function` to `icmp_rcv`, then to `udp_sendmsg`.

## Words to own

- **FIB**: forwarding information base, the routing table.
- **longest-prefix match**: the most specific matching route wins.
- **LC-trie**: level-compressed trie used for IPv4 FIB lookups.
- **ARP**: maps an IPv4 address to a MAC address on the local link.
- **ICMP**: control messages: echo, unreachable, time exceeded.
- **MTU / PMTU discovery**: largest packet size on a path; learned via ICMP.
- **TTL**: hop limit decremented by each router.
- **socket family / protocol**: `AF_INET` + `SOCK_DGRAM` = UDP.

## Answer in your notes

1. In `ip_rcv()`, what is checked before the packet reaches the routing decision?
2. Which netfilter hooks does a locally delivered packet pass, and which does a
   forwarded one pass?
3. What changes in `/proc/net/snmp` after one `ping 127.0.0.1`?
