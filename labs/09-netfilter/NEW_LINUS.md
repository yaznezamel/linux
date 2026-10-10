# Lab 09: netfilter

Week 9 (Dec 7-13). Read first:
[`net/ipv4/NEW_LINUS.md`](../../net/ipv4/NEW_LINUS.md),
[`net/netfilter/NEW_LINUS.md`](../../net/netfilter/NEW_LINUS.md)

## Goal

Hook into the IPv4 stack: count ICMP echo requests arriving at this machine and
drop every second one. This is what a firewall rule does, minus the rule engine.

## You write

- `icmp_counter.c`, `Makefile` (module)
- `udp_echo.c` (user space, C review)

## Steps

1. Fill in a `struct nf_hook_ops`: your hook function, `pf = NFPROTO_IPV4`,
   `hooknum = NF_INET_LOCAL_IN`, `priority = NF_IP_PRI_FIRST`.
2. Register it in the initial network namespace with
   `nf_register_net_hook(&init_net, &ops)`; unregister on exit.
3. In the hook: get the IP header with `ip_hdr(skb)`. If the protocol is ICMP,
   get the ICMP header with `icmp_hdr(skb)`; if its type is `ICMP_ECHO`,
   increment a counter and return `NF_DROP` for every second one. Return
   `NF_ACCEPT` for everything else.
4. Test with `ping -c 10 127.0.0.1`: expect 50% packet loss. Print the counter
   on unload.
5. Change the hook to `NF_INET_LOCAL_OUT`. Does the result change? Explain
   which hooks a ping to `127.0.0.1` passes through, in order.
6. Trace the drop: `echo 1 > /sys/kernel/tracing/events/skb/kfree_skb/enable`,
   ping once, and find your drop reason in the trace.

## Done when

- [ ] 50% loss on loopback ping with the module loaded, 0% after unloading.
- [ ] The counter matches the number of echo requests sent.
- [ ] You can name the hooks a loopback ping crosses, in order.

## Hints

- `include/linux/netfilter.h`: `struct nf_hook_ops`, `nf_register_net_hook()`
- `include/uapi/linux/netfilter_ipv4.h`: `NF_IP_PRI_FIRST`
- `include/linux/ip.h`: `ip_hdr()`; `include/linux/icmp.h`: `icmp_hdr()`;
  `include/uapi/linux/icmp.h`: `ICMP_ECHO`
- Use an `atomic_t` for the counter: hooks run in softirq context on any CPU.

## C review: sockets (Beej chapters 5-6)

`udp_echo.c`: a UDP echo server and client in one file (`./udp_echo server 9000`
and `./udp_echo client 127.0.0.1 9000 hello`). Next week you will trace its
`sendto()` with bpftrace.
