# Lab 08: mynic

Week 8 (Nov 30-Dec 6). Read first:
[`drivers/net/NEW_LINUS.md`](../../drivers/net/NEW_LINUS.md),
[`net/core/NEW_LINUS.md`](../../net/core/NEW_LINUS.md),
[`docs/networking/01-dummy-driver.md`](../../docs/networking/01-dummy-driver.md)

## Goal

Your own network device driver: a virtual NIC that counts what the stack sends
it, by protocol.

## You write

- `mynic.c`, `Makefile` (module)
- `parse.c` (user space, C review)

## Steps

1. Do the `dummy.c` walkthrough and trace in `docs/networking/01-dummy-driver.md` first.
2. Write `mynic.c` from scratch, using `dummy.c` as the pattern, not as a copy:
   `rtnl_link_ops` with `.kind = "mynic"`, a setup function, a
   `net_device_ops` with `.ndo_start_xmit`.
3. Keep counters in the device's private area (`alloc_netdev()` size argument,
   `netdev_priv()`): IPv4, IPv6, ARP, other, by `skb->protocol`.
4. Expose the counters: print them in the exit function, or (better) create a
   debugfs file per counter.
5. Don't set `IFF_NOARP`, so you can watch ARP requests arrive.
6. Test:
   ```sh
   ip link add mynic0 type mynic
   ip link set mynic0 up
   ip addr add 10.9.0.1/24 dev mynic0
   ping -c 3 -W 1 10.9.0.2          # ARP for 10.9.0.2 goes out first
   ip -6 addr show dev mynic0       # IPv6 link-local traffic counts too
   ```
7. Trace your transmit function with the `func_stack_trace` method from the
   dummy walkthrough and compare the stack with `dummy_xmit()`'s.

## Done when

- [ ] `ip link add ... type mynic` works and `ip -d link` shows `mynic`.
- [ ] The ARP counter goes up on the first ping, IPv4 does not (no reply ever
      comes, so no IPv4 packet is ever sent). Explain why.
- [ ] Unload with the device still present: no leak, no warning.

## Hints

- `include/linux/netdevice.h`, `include/linux/etherdevice.h` (`ether_setup()`,
  `eth_hw_addr_random()`), `include/net/rtnetlink.h` (`rtnl_link_register()`)
- `include/uapi/linux/if_ether.h`: `ETH_P_IP`, `ETH_P_IPV6`, `ETH_P_ARP`
- `skb->protocol` is in network byte order: compare with `htons(ETH_P_IP)`.

## C review: bits and byte order (K&R 2.9, 6.9)

`parse.c`: hard-code the bytes of one Ethernet + IPv4 + ICMP echo frame (copy
them from `tcpdump -xx` on your host) into an array, overlay packed structs,
and print source and destination MAC and IP, TTL and protocol. Use
`ntohs()`/`ntohl()` where needed.
